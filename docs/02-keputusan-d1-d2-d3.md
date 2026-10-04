# 02 — Keputusan D1, D2, D3

> Status: **D1 & D3 final. D2 = rekomendasi, menunggu persetujuan.**

---

## D1 — Deposit Gating · **FINAL**

**Keputusan:** agency wajib memiliki saldo mengendap positif sebagai jaminan sebelum jadwal COD boleh berjalan.

**Konsekuensi:**
- Konsep `credit_limit` (saldo boleh minus) **dibuang**. Tidak ada kredit dari platform ke agency di MVP.
- Legacy punya kolom `agency_policies.credit_limit` — **tidak dipakai** dan **tidak diimplementasikan** (kolom hidup, logika mati; terkonfirmasi di ekstraksi).
- Setiap agen harus punya jalan untuk **mengisi** deposit, dan jalan untuk **menariknya kembali** saat tidak ada jaminan aktif.

---

## D2 — Formula & Waktu Deposit · **REKOMENDASI**

### Kondisi legacy (untuk dihindari)

| Aspek | Legacy | Masalah |
|---|---|---|
| Basis | `route.cod_min_deposit` = **Rp 500.000 flat** per jadwal | Tidak terkait kapasitas maupun pendapatan platform yang berisiko |
| Jadwal PP | Di-hold **2×** | Jaminan digandakan tanpa alasan bisnis |
| Penyimpanan | Kolom `cod_hold_balance` | Tidak bisa menjawab "berapa yang di-hold untuk jadwal #42?" |
| Pelepasan | `releaseCodDeposit()` hanya membebaskan klaim, **uang tidak kembali ke `available_balance`** | Deposit **terkunci permanen** — tidak bisa ditarik sama sekali (`WithdrawalService` hanya memakai `available_balance`) |
| Aritmetika | Deposit 500rb + 1 jadwal COD ⇒ sisa 0 ⇒ **semua booking COD ditolak** | Praktiknya butuh ≥ 2× nominal |
| Idempotency | `releaseCodDeposit()` tanpa pengecekan status | **Double release** — `finish()` tidak menonaktifkan jadwal, cron 5 menit melepas hold berulang |

### Rekomendasi

**1. Basis perhitungan = eksposur platform, bukan harga tiket.**

Yang berisiko hilang saat COD adalah **pendapatan platform** (biaya layanan + biaya platform + PPN), karena uangnya dipegang supir/agency. Harga tiketnya sendiri bukan milik platform.

```
fee_platform_per_kursi = service_fee + platform_fee + PPN
hold_penuh(schedule)   = kapasitas_kursi × fee_platform_per_kursi
```

Ini persis rumusan yang kamu sebut: **kapasitas seat × total komisi platform.**

**2. Tapi jangan hold penuh di muka — pakai hold progresif.**

Hold penuh = seluruh kapasitas × fee akan **mematikan likuiditas agency kecil**. Padahal eksposur sebenarnya menyusut seiring tiket terjual dan disetor.

```
hold_aktif(schedule) = (kapasitas − kursi_yang_komisinya_sudah_disetor) × fee_platform_per_kursi
```

- **Saat publish:** hold = kapasitas penuh (worst case, aman)
- **Setiap settlement:** kursi yang komisinya sudah ditarik → hold-nya dilepas sebesar 1 kursi
- **Saat jadwal selesai & ter-verifikasi:** hold = 0

Keuntungan: aman di awal, ringan di akhir, dan agency tidak perlu menahan modal 2× seperti legacy.

**3. Waktu evaluasi (kapan dihitung ulang):**

| Pemicu | Aksi |
|---|---|
| Agency mengaktifkan COD pada jadwal | Hitung `hold_penuh`, gate: `deposit_tersedia ≥ hold_penuh` |
| Kapasitas berubah (ganti `seat_layout`) | Hitung ulang; jika kurang → blokir penjualan COD baru |
| Harga / `commission_override` berubah | Hitung ulang (fee per kursi berubah) |
| Setiap settlement diverifikasi | Lepas hold per kursi yang disetor |
| Agency menarik saldo | Gate: `deposit − hold_aktif` tetap ≥ 0 |
| Jadwal dibatalkan | Lepas seluruh hold jadwal itu |

**4. Gate harus spesifik, jangan memblokir berlebihan.**

Kalau `deposit_tersedia < hold_penuh`:
- **Blokir penambahan** kursi/booking COD baru
- **Jangan batalkan** tiket yang sudah terjual secara diam-diam
- Tandai jadwal dengan status risiko (`ok` / `warning` / `blocked`) dan beri tahu agency + ops

**5. Desain ledger — ini yang paling penting untuk diubah dari legacy.**

Hold **tidak boleh** jadi kolom saldo. Harus jadi **record** yang diaudit:

```
wallet_holds
├── id
├── account_id          (pemilik saldo)
├── source_type         ('schedule' | 'rental' | 'booking')
├── source_id           (jadwal #42)
├── amount              (nominal yang di-hold)
├── status              ('active' | 'released' | 'consumed' | 'expired')
├── reason              ('cod_collateral' | 'ots_deposit')
├── released_at
├── ledger_entry_id     (jejak ke mutasi)
└── unique(source_type, source_id, reason)   ← IDEMPOTENCY
```

`unique(source_type, source_id, reason)` inilah yang **mencegah double release** yang terjadi di legacy.

**6. Tiga saldo yang harus dipisah tegas:**

```
available       = uang bebas, boleh ditarik
held            = uang terkunci sebagai jaminan (= SUM hold aktif)
withdrawable    = available − (hold_aktif − available)
```

Aturan: **penarikan hanya dari `available`, dan tidak boleh membuat `held > available`.**

**7. Padanan industri** (untuk justifikasi ke investor):
- **IATA** — jaminan bank/garansi untuk agen travel, disesuaikan volume penjualan
- **Hotel** — deposit insidental, dilepas saat check-out
- **Gojek/Grab** — deposit saldo driver untuk transaksi tunai

Ketiganya: jaminan disesuaikan **eksposur**, bukan angka tetap.

---

## D3 — Model Pendapatan Platform · **FINAL + KOREKSI PENTING**

### Koreksi: tidak ada komisi yang dipotong dari agency

Dari `PricingService.php` — kode menyebut dirinya **"Model B (income model GoMad)"**:

> Agency menerima **100% nilai fare** (`agencyNet = base − promo`). **TIDAK PERNAH** dipotong komisi platform dari harga agency.
> Income platform = **fee layanan + fee platform** yang dibebankan ke customer **di atas** harga agency.
> Khusus channel **CASH (warung)**: komisi warung 2% dibayar **dari porsi income platform**, bukan dari agency.

```
total_dibayar_customer = agency_net + (service_fee + platform_fee) [+ warung_commission, jika cash]
```

Jadi `commission_rate = 5%` di config adalah **jalur LEGACY**, dan komentar di kode menyatakan pemanggil produksi wajib memakai Model B. **Perlu diverifikasi** apakah benar sudah tidak ada pemanggil legacy.

### Struktur pendapatan platform

| Komponen | Ditanggung | Masuk ke |
|---|---|---|
| `service_fee` | Customer (di atas fare) | Pendapatan platform |
| `platform_fee` | Customer (di atas fare) | Pendapatan platform |
| **PPN 11%** | Dihitung dari **total komisi** (= fee platform), **bukan** dari total biaya transaksi | ⚠️ **Utang pajak** |
| `warung_commission` 2% | Dipotong dari pendapatan platform, **bukan** dari agency | Biaya platform |

### ⚠️ Dua catatan penting

**1. PPN bukan pendapatan.** PPN adalah **kewajiban pajak** — platform memungut dari customer lalu menyetorkan ke negara. Di ledger **wajib** dipisah:

```
Revenue: Service Fee        → pendapatan
Revenue: Platform Fee       → pendapatan
Tax Payable: PPN Keluaran   → utang, BUKAN pendapatan
```

Kalau PPN dicatat sebagai pendapatan, laporan laba salah **dan** pelaporan pajak tidak bisa dilakukan.

**2. PPN belum ada sama sekali di legacy.** Ekstraksi menyatakan eksplisit: **"PPN/pajak: TIDAK DITEMUKAN"** — tidak ada perhitungan, tidak ada kolom, tidak ada akun. Ini **fitur baru** di Core, bukan migrasi.

**Bonus temuan:** pemroses **refund tunai untuk warung tidak ada** — status `refund_pending` menggantung tanpa jalan keluar. Ini harus dirancang di Core.
