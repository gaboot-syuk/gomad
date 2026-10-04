# P1 — Sangat membantu

> Bukan blocker, tapi **mempercepat & mengurangi salah tafsir** secara signifikan.
> Urutan di bawah disusun dari yang paling menghemat waktu.

---

## P1-7 · Akses demo per role

**Yang dibutuhkan:** login customer, agency, driver, admin.

**Kenapa:** Paling efisien — bisa menelusuri alur nyata, bukan tanya-jawab bolak-balik.

**Cara terbaik:** satu akun per role di **satu lingkungan yang stabil** (staging), plus catatan "apa yang sudah/sedang di kerjakan di lingkungan ini".

**Di repo ini:** ada dua lingkungan demo, tapi keduanya **bukan** lingkungan uji yang ideal:
- `demo-static/` — ±258 halaman HTML mockup (`demo.gomad.id`). Bagus untuk melihat tampilan, **tidak ada data/logika nyata**.
- Clone Laravel dengan SQLite read-only (`demo.sqlite`) — read-only, dan ada jebakan: login dengan "remember me" memicu error 500.

Kredensial uji ada di **catatan internal** (`memories/gomad-notes.md`). Jangan disalin ke dokumen ini.

**Yang masih kurang:**
- ⚠️ Lingkungan demo yang **bisa diubah** (untuk mengetes alur end-to-end, bukan cuma melihat).
- Catatan status tiap lingkungan: mana produksi, mana staging, mana arsip.

---

## P1-8 · Contoh payload nyata

**Yang dibutuhkan:** booking, callback Midtrans, settlement, Koin COD.

**Kenapa:** Menentukan bentuk kontrak API baru.

**Format ideal:** request + response **asli** (tersalin apa adanya) untuk 4 alur itu, termasuk kasus error (422 validasi, 402/409 saat jadwal penuh, dsb).

**Cara cepat dapat:**
- Ambil dari log aplikasi (`storage/logs/`) untuk payload webhook Midtrans.
- Rekam lewat browser DevTools / proxy untuk payload booking.
- Kalau tidak ada rekaman: jalankan satu booking di lingkungan demo, lalu salin.

**Yang masih kurang:** ⚠️ Belum ada rekaman sama sekali di repo. Ini murni harus di-*capture* manual.

---

## P1-9 · Detail Midtrans

**Yang dibutuhkan:** merchant atas nama entitas apa, produk mana, penanganan callback & idempotency.

**Kenapa:** Langsung menyentuh `Core/Settlement`.

**Yang sudah diketahui:** ada 4 webhook — **booking**, **disbursement/withdrawal**, **settlement**, **topup**. Status saat ini: **sandbox**.

**Yang masih kurang:**
- ⚠️ **Entitas legal merchant** (PT/CV mana) — belum diketahui dari repo.
- ⚠️ **Produk Midtrans** yang dipakai (Snap? Core API? Iris untuk disbursement?) — perlu konfirmasi.
- ⚠️ **Idempotency.** Bagaimana callback duplikat ditangani? Apakah ada pengecekan `order_id`/status sebelum update saldo? **Ini kritik untuk `Core/Settlement`** — kalau sistem lama tidak menangani ini, Core baru harus menambahkannya.
- Signature verification: apakah diverifikasi, atau hanya percaya payload?
- Shadow testing: bagaimana membedakan transaksi sandbox vs produksi di database?

**Cara cepat dapat:** kirim salinan halaman *Settings → Access Keys & webhook config*, plus kode controller/handler callback.

---

## P1-10 · Mekanisme payout ke agency saat ini

**Yang dibutuhkan:** manual transfer atau disbursement API? Ada laporan rekonsiliasi?

**Kenapa:** Menentukan desain pencairan & saldo mengendap.

**Yang sudah terbaca dari repo:** `WithdrawalService` + `SettlementService`, dengan aturan: minimum penarikan, ada biaya tetap, **auto-approve untuk nominal kecil** (langsung kredit), **nominal besar butuh approval admin**. Settlement warung di-**generate otomatis tiap Senin** (cron), dengan `amount_to_settle = total - komisi`, berstatus `pending → paid → verified`.

**Yang masih kurang (krusial):**
- ⚠️ **Apakah uang benar-benar berpindah?** Sistem lama memakai Midtrans **sandbox** — jadi kemungkinan besar pencairan masih **simulasi**. Perlu jawaban pasti: siapa yang mentransfer manual, lewat apa (bank/e-wallet), dan bukti transfernya dicatat di mana.
- ⚠️ **Rekonsiliasi.** Ada tidak proses mencocokkan uang keluar-masuk? Kalau tidak ada, ini **kerjaan baru** di Core.
- **Saldo mengendap** (`pending`, `cod_hold_balance`, `deposit`) — kapan dilepas, oleh siapa, audit trail-nya di mana.
- **Perlakuan pajak** (PPh/PKP) kalau ada — belum terlihat di kode.

---

## P1-11 · Dokumentasi yang sudah ada

**Yang dibutuhkan:** `Panduan Customer`, `Panduan Agency`, `Panduan Driver`, `Panduan Warung`.

**Kenapa:** Sudah setengah jalan menuju dokumen domain.

**Di repo ini:** ⚠️ **Empat panduan itu tidak ditemukan.** Yang ada:

| Lokasi | Isi |
|---|---|
| `gomad-docs/Flow-Transaksi.pdf` | Dokumen alur transaksi |
| `gomad-docs/flow-report/`, `flow-shots/` | Laporan & tangkapan layar alur |
| `gomad-docs/investor/` | 17 file — market research, business model, traction, proyeksi keuangan, pitch deck (A/B/C) |
| `gomad-docs/promotion/` | 8 file — paket produk/bisnis/marketing/finansial/operasional |
| `memories/` | Catatan teknis hasil eksplorasi (arsitektur, alur, konvensi) |

**Catatan:** `gomad-docs/` bersifat **arsip**, tidak terhubung ke kode — jadi jangan dijadikan sumber kebenaran untuk perilaku sistem.

**Yang masih kurang:** panduan operasional per role memang belum ada. Ini bisa diturunkan dari kode kalau tidak ditemukan.

---

## P1-12 · Contoh data produksi

**Yang dibutuhkan:** 1 jadwal, 1 armada + denah kursi, 1 rute multi-segmen, 1 booking penuh. Plus **jumlah baris per tabel**.

**Kenapa:** Sekaligus mengukur apakah angka "27++ booking" itu nyata atau seed.

**Format ideal:**
- 4 contoh data sebagai CSV/JSON, **nama & kontak dianonimkan**.
- Jumlah baris semua tabel (`SELECT COUNT(*)` per tabel → satu CSV).

**Cara cepat dapat:**
```sql
SELECT table_name, table_rows
FROM information_schema.tables
WHERE table_schema = '<dbname>';
```
(`table_rows` di InnoDB adalah estimasi — untuk tabel inti sebaiknya `SELECT COUNT(*)` langsung.)

**Petunjuk yang sudah ada:**
- Di `memories/` tercatat **2 booking nyata** (#9 Sumenep→Surabaya, menunggu pembayaran; #2 Surabaya→Malang, sudah dibayar) — ini bisa jadi titik awal verifikasi.
- Ada catatan **konflik data kursi supir**: denah kursi di DB lokal berbeda dengan produksi (`A1` vs `A2`). Ini bukti bahwa data produksi ≠ data lokal — dan alasan kenapa P1-12 penting.

**Yang masih kurang:** ⚠️ Jumlah baris per tabel belum pernah diambil. Perlu akses produksi.

---

## P1-13 · Daftar masalah yang kamu sudah tahu

**Yang dibutuhkan:** apa yang salah/terbatas di build sekarang.

**Kenapa:** Menghindari mengulang kesalahan yang sama.

**Di repo ini:** sudah tercatat, tapi **tersebar** di `memories/`. Yang sudah terdokumentasi:

| Area | Masalah |
|---|---|
| Mobile login | Role agency/admin/payment_agent ditolak app — pembatasan disengaja, tapi pesan & UX-nya kasar |
| Kapasitas jadwal | Kapasitas dihitung dari `sum(total_passengers)` booking non-cancelled → duplikat kursi bisa membuat jadwal "penuh" padahal tidak |
| Data kursi | `driver_seat` berbeda antara DB lokal dan produksi → auto-assign kursi berisiko |
| Error API | Backend memakai key `data` untuk error 422, sementara klien membaca `errors` → pesan asli tertelan jadi error generik |
| Demo | Login "remember me" di demo SQLite read-only → 500 |
| Web session | Session web mudah kedaluwarsa; beberapa halaman sesekali 500 sesaat |
| Observabilitas | Error 422/500 tidak terlihat dari sisi klien — sulit didiagnosis tanpa akses log |

**Yang masih kurang:**
- **Konsolidasi.** Daftar ini perlu dipindahkan dari catatan eksplorasi ke satu dokumen "known issues" resmi, ditandai mana yang *bug* vs *keterbatasan desain* vs *utang teknis*.
- **Prioritas dampak**: mana yang menyebabkan kerugian uang/data vs hanya mengganggu UX.
- **Daftar masalah dari sisi operasional** (bukan teknis) — mis. driver lupa konfirmasi, warung tidak setor tepat waktu.
