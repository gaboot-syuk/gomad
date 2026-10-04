# 03 — Keputusan Turunan dari Hasil Ekstraksi

> Temuan yang menuntut keputusan struktural. Semua sudah punya rekomendasi.
> Sumber: `docs/legacy/extract-money-risk.md`, `docs/legacy/extract-siklus-booking.md`

---

## D9 — Kunci kursi & TTL kursi

**Kondisi legacy:** tidak ada kunci eksplisit. Tidak ada tabel reservasi, tidak ada `seat_locked_until`.

Kursi "terkunci" hanya karena baris `booking_passengers` dibuat sejak status `pending`. TTL efektifnya = kedaluwarsa pembayaran:

| Kanal | TTL efektif |
|---|---|
| Midtrans | 30 menit |
| Cash / Warung | 24 jam |
| **COD** | **tanpa batas** ⚠️ |

Cron expire berjalan tiap 5 menit, jadi kursi nyangkut hingga TTL + jeda cron.

**Rekomendasi:** buat **kunci kursi eksplisit dengan TTL** di Core, terpisah dari booking.
- TTL 15 menit untuk pembayaran digital
- TTL khusus untuk COD — **wajib ada**, ini lubang terbesar
- Kursi dilepas otomatis oleh scheduler, bukan mengandalkan booking pending

**Alasan:** kursi adalah inventori terbatas. Menahan inventori tanpa batas waktu = kehilangan penjualan yang tidak terlihat.

---

## D10 — Cutoff penjualan

**Kondisi legacy:** tidak ada. Booking ditolak hanya kalau jam keberangkatan sudah `isPast()` → **bisa pesan 1 menit sebelum berangkat**. Config `schedule_min_days_before = 30` ternyata **config mati**.

**Rekomendasi:** cutoff default **60–120 menit sebelum keberangkatan**, configurable per agency.
**Alasan:** travel door-to-door butuh waktu susun rute jemput + manifest penumpang. Tanpa cutoff, supir bisa menerima penumpang baru saat sudah berjalan.

---

## D11 — Kapasitas: per jadwal atau per segmen?

**Kondisi legacy:** kapasitas dihitung **per jadwal**, bukan per segmen. Akibatnya penumpang A→B dan C→D **saling mengurangi kursi yang sama**.

Untuk travel multi-stop/door-to-door ini salah secara komersial: kursi yang ditinggalkan penumpang di titik B seharusnya bisa dijual ke penumpang C→D. Efeknya **kehilangan pendapatan**, bukan risiko keselamatan.

**Rekomendasi:** MVP tetap **per jadwal** (sederhana & tidak pernah menjual kursi fisik dua kali), tetapi model kursi dirancang **`kursi × segmen`** sejak awal agar alokasi per segmen bisa dinyalakan tanpa migrasi besar.
**Trade-off yang harus kamu sadari:** sampai fitur ini nyala, kamu kehilangan sebagian penjualan rute pendek di tengah.

---

## D12 — Siapa menanggung diskon promo?

**Kondisi legacy:** kolom `cost_bearer`, `platform_share_percent`, `agency_share_percent` **ada tapi tidak pernah dibaca**. Praktiknya **agency yang menanggung** (`agency_net = base − discount`), sementara platform tetap menerima fee penuh.

Ironisnya, promo **referral** dibuat dengan `cost_bearer = 'platform'` dan share 100% — tapi tetap memotong agency. Jadi ini **bug komersial**, bukan sekadar desain.

**Rekomendasi:** hidupkan `cost_bearer`:
- Promo akuisisi/referral (kepentingan platform) → **platform** yang menanggung
- Promo buatan agency sendiri → **agency** yang menanggung
- Pastikan `platform_revenue` benar-benar berkurang saat platform menanggung, dan `agency_net` tetap penuh

**Alasan:** kalau agency merasa diskon platform selalu dipotong dari mereka, mereka akan berhenti ikut promo — dan mesin akuisisi customer mati.

---

## D13 — Koin COD (`cod_credit_balance`) · **FINAL: DIHAPUS TOTAL**

**Keputusan:** Koin COD dihapus total dari GoMad baru. Tidak ada saldo virtual, tidak ada mata uang bayangan. Jika butuh insentif, pakai diskon top-up deposit atau tier fee.

**Kondisi legacy:** Koin COD adalah "saldo virtual untuk hold COD". Rumus kapasitas COD jadi:

```
kapasitas_cod = deposit_balance + cod_credit_balance − cod_hold_balance
```

**Masalah:** koin tidak punya padanan uang nyata. Ada tiga kantong uang dengan aturan berbeda, dan satu di antaranya fiktif. Hasilnya sulit diaudit dan sulit dijelaskan ke investor/auditor.

**Rekomendasi: hapus Koin COD.** Ganti dengan satu model bersih:

```
saldo (available / held)  +  hold per jadwal (record, bukan kolom)
```

**Alasan:** kalau koin adalah cara memberi insentif, insentif yang lebih baik adalah **diskon top-up deposit** atau **tier komisi** — bukan mata uang bayangan yang menyulitkan pembukuan.

---

## D14 — Koin & fitur yang harus dinyatakan mati

Dari ekstraksi, kolom-kolom ini **ada di DB tapi logikanya tidak pernah jalan**. Harus diputuskan: dihidupkan atau dibuang.

| Kolom | Status | Rekomendasi |
|---|---|---|
| `agency_policies.credit_limit` | Mati | **Buang** — sudah diputuskan D1 = deposit gating |
| `agency_policies.cod_daily_limit` | Mati | Hidupkan (batas risiko per hari) |
| `agency_policies.cod_max_per_booking` | Mati | Hidupkan (batas risiko per booking) |
| `agency_policies.commission_override` | Mati | Hidupkan (Model B: override fee per agency) |
| `schedules.transfer_fee_per_passenger` | Selalu 0 | Perjelas atau buang |
| `schedules.max_transfer_fee_percent` | Selalu 0 | Perjelas atau buang |
| `agency_policies.allow_cod_without_deposit` | Mati | **Buang** — bertentangan dengan D1 |

---

## D15 — Berapa banyak utang teknis yang diperbaiki di MVP?

Temuan ekstraksi memuat **25 temuan berisiko (R1–R25)** di bagian uang saja, termasuk:

- 🔴 **Idempotency disbursement withdrawal bolong** — tanpa verifikasi signature & tanpa cek status ⇒ callback `failed` berulang = **pengembalian dana berulang**
- 🔴 **Double release hold COD** oleh cron expire
- 🔴 **Tidak ada unique constraint kursi** + bug auto-assign menulis ke salinan `foreach` ⇒ kursi dobel
- 🟠 **`confirmCod` Web vs API menghasilkan status akhir booking berbeda**
- 🟠 **Refund tunai warung tidak punya pemroses** — `refund_pending` menggantung
- 🟠 **Tidak ada PPN** sama sekali di kode

**Rekomendasi:** semua 🔴 **wajib** dibereskan di MVP, karena ini menyangkut uang langsung dan akan terulang di Core baru kalau tidak diperbaiki secara sadar (bukan "nanti ikut ter-refactor"). Yang 🟠 bisa masuk backlog dengan alasan tertulis.

**Pertanyaan yang perlu kamu jawab:** apakah 25 temuan itu sudah cukup, atau kamu mau saya telusuri **model & controller** juga (bukan hanya service) untuk memastikan tidak ada logika aturan bisnis yang tersembunyi di controller?
