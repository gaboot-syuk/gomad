# Blueprint · Rencana Fase

> **Peta eksekusi dari awal sampai semuanya bisa diakses.**
> **Tanpa tanggal** — tapi setiap fase **wajib production-ready** sebelum lanjut.
> Prinsip: *web dulu untuk semua peran*, sambil menjaga kesiapan mobile native.

---

## Cara membaca dokumen ini

| Istilah | Arti |
|---|---|
| **Kriteria lulus** | Syarat yang harus terbukti — bukan "sudah dibuat" |
| **Production-ready** | Termasuk tes · penanganan error · keamanan · siap deploy |
| ⭐ **Verifikasi live** | Fase ditutup dengan **menjalankan aplikasi di Docker dan menelusuri alurnya di browser preview** — bukan hanya tes otomatis. Lihat `09-lingkungan-dan-verifikasi-live.md` |
| **Kesiapan mobile** | Hal yang dijaga agar mobile nanti tidak perlu membongkar apa pun |
| **Akses** | Yang sudah bisa dibuka & dipakai setelah fase selesai |

---

## Ringkasan fase

| Fase | Nama | Akses setelah selesai |
|---|---|---|
| **0** | Fondasi & keputusan | Staging kosong ber-URL |
| **1** | Core — Identity & Access | Login & otorisasi semua peran |
| **2** | Core — Ledger & Settlement | Ledger teruji (belum ada transaksi nyata) |
| **3** | Core — Payment & Notification | Uang masuk & keluar terbukti |
| **4** | Mad Warung inti | **Satu order nyata**, warung → grosir |
| **5** | Mad Trans inti | **Satu perjalanan nyata**, akhir ke akhir |
| **6** | Mad Trans lanjutan | Rental · charter · multi-stop · transfer penumpang |
| **7** | Keterlibatan | Promo · referral · level & misi |
| **8** | Pengerasan & rilis | Rilis lengkap di domain publik |
| **9** | Persiapan mobile native | Kontrak API & auth siap untuk Flutter |

---

## FASE 0 — Fondasi & lingkungan kerja

**Tujuan:** menyiapkan tempat kerja yang bisa dijalankan dan dihancurkan kapan saja, lalu membuktikan alur verifikasi live.

> Keputusan bisnis sudah selesai. Hosting staging Render free tier sengaja
> ditunda sampai seluruh alur lokal berjalan lancar.

**Langkah — kerjakan berurutan**

| # | Langkah | Catatan |
|---|---|---|
| 1 | **Bersihkan sisa lama** — container `gomad-app-1` & `gomad-database-1`, dan folder `.smoke-test` | Namanya akan bentrok dengan project baru |
| 2 | **Repo `api/`** — Laravel + `git init` | Repo pertama |
| 3 | **Repo `mobile/`** — `git init`, isi kosong dulu | Repo kedua |
| 4 | **Stack Docker** di `api/docker/compose.yaml`: nginx · app (php-fpm) · queue · scheduler · Aiven MySQL via TLS · redis · mailpit · vite | Port: **8200** · **6380** · **8125** · **5273** |
| 5 | **Jalankan & buktikan** — `compose up`, halaman dibuka di **browser preview editor** | Menutup syarat verifikasi live pertama |
| 6 | **Perintah harian** — `compose up/down` · `artisan migrate --seed` · `artisan test` · `app:reset-demo` | tes SQLite terisolasi; reset demo ditolak untuk database remote |
| 7 | **CI dasar** — lint · tes · uji arsitektur (kerangka) · migrasi dari nol | Gagal = blokir |
| 8 | **Backup terjadwal** + **uji pemulihan** | Backup yang belum dipulihkan belum bisa disebut backup |
| 9 | **Kontrak API v0** — kerangka · cara generate · cara memberi sinyal ke repo `mobile/` | Karena dua repo terpisah |

**Kriteria lulus**
- [x] Container GoMad lama dibersihkan sebelum stack baru dinyalakan
- [x] `compose up` menyalakan seluruh stack, dan halaman terbuka di **browser preview editor**
- [x] Migrasi bersih + seed berjalan setelah izin eksplisit untuk menghapus schema Aiven lama; `app:reset-demo` menolak database remote sebagai pengaman
- [x] CI remote hijau pada commit integrasi; tes lokal, formatting, build aset, migrasi bersih + seed juga lulus
- [x] Backup terjadwal ke penyimpanan lokal dan R2; pemulihan lokal serta pemulihan nyata dari objek R2 sudah diuji
- [ ] Kontrak API punya tempat resmi dan cara generate; penerima `repository_dispatch` mobile lulus uji remote. Pengiriman otomatis dari API perlu secret GitHub Actions sebelum dapat dinyatakan selesai
- [x] Prosedur rilis ditulis, walau rilis pertama belum terjadi

**Kesiapan mobile:** struktur responsif disiapkan sejak awal (satu layout, banyak ukuran)

**Akses:** aplikasi lokal di `http://localhost:8200` lulus verifikasi live; staging Render baru disiapkan setelah seluruh alur local berjalan lancar.

---

## FASE 1 — Core: Identity & Access

**Tujuan:** satu identitas untuk semua peran, dengan otorisasi berbasis aksi.

**Isi**
- Skema identitas: satu orang → banyak peran (prinsip *Unified User Identity*)
- Peran: **Pemilik Usaha · Staff · Pihak Luar · Admin** — satu model untuk semua vertical
- **Otorisasi berbasis aksi**, bukan sekadar cek role di middleware
- Multi-outlet: satu pemilik → banyak usaha; staff terikat ke usaha (siap banyak-ke-banyak)
- Auth ganda: **session untuk web**, **token untuk mobile** (Sanctum)
- Undangan & onboarding pengguna (pemilik mendaftarkan penjaganya)

**Kriteria lulus**
- [ ] Tes: satu orang dengan dua peran bisa mengakses keduanya tanpa akun kedua
- [ ] Tes: staff tidak bisa mengakses data usaha lain
- [ ] Tes: otorisasi menolak aksi yang tidak diizinkan — di **satu** tempat, bukan tersebar
- [ ] Token mobile bisa dipakai dari luar sesi web
- [ ] Tidak ada pengecekan role yang tersebar di controller

**Kesiapan mobile:** alur login token sudah dibuktikan, bukan dirancang saja

**Akses:** login, kelola pengguna, undang staff

---

## FASE 2 — Core: Ledger & Settlement

**Tujuan:** mesin uang yang bisa diaudit dan direkonsiliasi.

**Isi**
- `ledger_accounts` · `ledger_transactions` · `ledger_entries` · `wallet_holds`
- **Double-entry** dengan `category`, `reference`, dan `balance_after` per akun
- Akun: `PLATFORM.*` · `AGENCY.AVAILABLE` · `AGENCY.COLLATERAL` · `GROSIR.AVAILABLE`
- **PPN sebagai akun kewajiban**, bukan pendapatan
- Idempotency di level ledger + reversal (tidak pernah menimpa entri)
- Hold sebagai *record* dengan kunci unik — bukan kolom saldo
- Settlement engine: per agency & per grosir

**Kriteria lulus**
- [ ] **Tes rekonsiliasi:** jumlah mutasi merekonstruksi saldo setiap akun
- [ ] **Tes callback duplikat:** dikirim 3× → saldo berubah **sekali**
- [ ] **Tes race condition:** dua permintaan bersamaan → satu berhasil
- [ ] **Tes reversal:** pembalikan tidak meninggalkan jejak ganda
- [ ] PPN tidak pernah masuk akun pendapatan
- [ ] Hold tidak bisa dilepas dua kali

**Kesiapan mobile:** — (murni backend)

**Akses:** belum ada UI; ledger terbukti lewat tes

---

## FASE 3 — Core: Payment & Notification

**Tujuan:** uang benar-benar masuk dan keluar.

**Isi**
- Payment gateway (Midtrans): pembayaran per transaksi, webhook, verifikasi signature
- **Fallback pembayaran** bila auto-charge gagal (QRIS/VA) — menjawab isu OTP/3DS
- Tokenisasi metode pembayaran (jangan pernah menyimpan data kartu)
- Pencairan: pengajuan → persetujuan → eksekusi → pembalikan bila gagal
- Settlement terjadwal per mitra
- Notifikasi WhatsApp + in-app, template per peran
- Buku besar operasional: rekonsiliasi harian bisa dijalankan manual

**Kriteria lulus**
- [ ] Pembayaran nyata (nominal kecil) berhasil di staging
- [ ] Webhook palsu **ditolak** tanpa signature valid
- [ ] Pencairan gagal **mengembalikan** dana dengan benar
- [ ] Rekonsiliasi manual cocok 100% dengan ledger
- [ ] Notifikasi sampai ke penerima yang tepat

**Kesiapan mobile:** endpoint pembayaran tidak bergantung pada sesi web

**Akses:** pembayaran, pencairan, notifikasi berjalan

---

## FASE 4 — Mad Warung inti (web)

**Tujuan:** satu order nyata, dari penjaga sampai barang diterima.

**Isi**
- **Portal Pemilik Warung**: multi-outlet · plafon penjaga · approval · jadwal shift · laporan
- **Halaman Penjaga** (web responsif): katalog · pesan · bayar · riwayat
- **Portal Grosir**: katalog · terima pesanan · tandai siap
- Alur order: plafon → approval → pesanan → grosir siapkan → antar/ambil → **konfirmasi terima** → settlement ke grosir
- Model pembayaran: penjaga menekan bayar, dana dari metode pembayaran milik pemilik

**Kriteria lulus**
- [ ] Satu order **nyata** oleh warung sungguhan, dari pesan sampai barang diterima
- [ ] Grosir menerima settlement-nya dengan benar
- [ ] Penjaga **tidak bisa** melewati plafon; approval pemilik bekerja dari jarak jauh
- [ ] Konfirmasi penerimaan tercatat dan bisa ditelusuri
- [ ] **Verifikasi live**: alur ditelusuri di browser preview, termasuk di layar kecil
- [ ] Uji dengan **1 warung + 1 grosir sungguhan** di **satu wilayah**

**Kesiapan mobile:** Penjaga memakai web di HP — jadi desainnya sudah teruji untuk layar kecil

**Akses:** Mad Warung dipakai pemilik, penjaga, dan grosir — lewat web

> 🎯 **Tonggak: pilot-able** — vertical utama bisa dipakai mitra sungguhan

---

## FASE 5 — Mad Trans inti (web)

**Tujuan:** satu perjalanan nyata, dari pesan sampai dana sampai ke agency.

**Isi**
- Travel: cari jadwal · pilih kursi · booking door-to-door · e-ticket · approval jadwal
- Portal Agency (web): kelola jadwal, armada, driver, terima pesanan
- **Dompet agency + pencairan**
- **COD + deposit gating** — termasuk pelepasan hold dua jalur (tertagih & void)
- Halaman driver (web responsif untuk sementara)

**Kriteria lulus**
- [ ] Satu perjalanan **nyata** oleh pengguna nyata, dari pesan sampai selesai
- [ ] Dana sampai ke saldo agency dan bisa dicairkan
- [ ] COD tanpa deposit cukup **ditolak di satu titik**, tanpa memblokir yang lain
- [ ] Kursi tidak bisa dobel, termasuk saat dua orang memesan bersamaan
- [ ] **Verifikasi live**: alur ditelusuri di browser preview
- [ ] Uji dengan **1 agency sungguhan** — bukan data contoh

**Kesiapan mobile:** semua alur sudah berbasis API, tanpa ketergantungan pada halaman web

**Akses:** Mad Trans bisa dipakai penumpang, agency, dan driver — lewat web

> Mad Trans hidup kembali di Core baru. Tonggak demo-able sudah dilewati di Fase 4.

---

## FASE 6 — Mad Trans lanjutan (web)

**Tujuan:** melengkapi Mad Trans sesuai keputusan "semuanya masuk".

**Isi**
- **Rental**: lepas kunci & dengan supir · kalender ketersediaan · verifikasi dokumen · siklus serah terima
- **Charter**
- **Multi-stop** — dengan **kapasitas per segmen** (perbaikan dari sistem lama)
- **Transfer penumpang antar jadwal** — biaya transfer harus benar-benar berfungsi
- **OTS** — perlakuan risiko seperti COD
- **Deposit penyewa** — ditagih, ditahan, dan dikembalikan dengan benar

**Kriteria lulus**
- [ ] Rental tidak bisa dobel-booking
- [ ] Multi-stop: kursi yang ditinggalkan penumpang di titik tengah **bisa dijual lagi**
- [ ] Transfer penumpang memindahkan kursi & uang dengan benar di **kedua** jadwal
- [ ] OTS gagal bayar menahan jaminan sesuai aturan
- [ ] Deposit penyewa kembali tepat waktu dan tercatat

**Kesiapan mobile:** —

**Akses:** seluruh moda Mad Trans tersedia

---

## FASE 7 — Keterlibatan

**Tujuan:** mesin retensi — dan uang diskon yang benar.

**Isi**
- **Promo** dengan `cost_bearer` yang benar-benar dihormati
- **Referral** — perbaikan dari sistem lama (dulu memotong agency padahal ditandai platform)
- **Level & Misi**

**Kriteria lulus**
- [ ] Promo platform **tidak** memotong agency — dibuktikan tes
- [ ] Promo agency memotong agency — dibuktikan tes
- [ ] Rincian diskon terlihat di settlement

**Akses:** promo & referral bisa dipakai

---

## FASE 8 — Pengerasan & rilis

**Tujuan:** dari *pilot-able* jadi *release-able*.

**Isi**
- **Feature freeze** — tidak ada fitur baru
- Audit menyeluruh alur uang · rekonsiliasi penuh · perbaikan temuan
- Keamanan: otorisasi, rate limit, validasi input, audit log
- Backup & **uji pemulihan sungguhan**
- Prosedur operasional tertulis: pencairan, sengketa, refund, kegagalan pengiriman
- Domain publik, monitoring, alerting
- Uji beban ringan

**Kriteria lulus**
- [ ] Rekonsiliasi penuh cocok, tanpa selisih 1 rupiah
- [ ] Uji pemulihan backup berhasil
- [ ] Tidak ada temuan 🔴 dari `docs/04-checklist-jangan-diulang.md` yang tersisa
- [ ] Prosedur operasional tersedia dan sudah dicoba
- [ ] Monitoring memberi tahu saat ada kegagalan uang

**Akses:** **Rilis lengkap** di domain publik

> 🎯 **Tonggak: release-able**

---

## FASE 9 — Persiapan mobile native

**Tujuan:** memastikan Flutter bisa dibangun tanpa membongkar apa pun.

**Isi**
- **Bekukan kontrak API** + dokumentasi lengkap
- Verifikasi auth token, upload berkas, dan notifikasi push dari luar web
- Petakan komponen web → widget Flutter (design system)
- Generator klien API untuk Dart
- Kerangka `mobile/` + satu alur pilot

**Kriteria lulus**
- [ ] Klien Dart bisa dibuat otomatis dari kontrak, tanpa ditulis manual
- [ ] Auth token & refresh bekerja dari aplikasi
- [ ] Satu alur pilot jalan di Android
- [ ] Tidak ada logika bisnis yang perlu diduplikasi di mobile

**Akses:** aplikasi native siap dibangun bertahap

---

## Pelacak Status

> **Diperbarui setiap sesi berakhir.** Ini yang dibaca sesi berikutnya.

| Fase | Status | Catatan |
|---|---|---|
| Konsep | ✅ Selesai | `docs/08` · `09` · `11` |
| Blueprint | ✅ **Selesai — 9 dari 9** | `blueprint/README.md` |
| 0 — Fondasi | 🔄 Dikerjakan | Migrasi/seed Aiven via TLS, backup/restore lokal↔R2, CI API remote, dan dispatch mobile nyata lulus; secret notifikasi otomatis belum disetel |
| 1 — Identity & Access | ⬜ Belum | |
| 2 — Ledger & Settlement | ⬜ Belum | |
| 3 — Payment & Notification | ⬜ Belum | |
| 4 — Mad Warung inti | ⬜ Belum | |
| 5 — Mad Trans inti | ⬜ Belum | |
| 6 — Mad Trans lanjutan | ⬜ Belum | |
| 7 — Keterlibatan | ⬜ Belum | |
| 8 — Pengerasan & rilis | ⬜ Belum | |
| 9 — Persiapan mobile | ⬜ Belum | |

**Sedang dikerjakan:** Menutup Fase 0 — menyiapkan secret notifikasi kontrak di GitHub; setelah alur local/CI lengkap, staging Render tetap langkah terakhir sesuai keputusan pengguna.
**Berikutnya:** Mulai Fase 1 setelah gerbang Fase 0 dan seluruh kriteria rilis-able terpenuhi.
**Blocker terbuka:** Pengguna perlu mengatur secret repo API `MOBILE_REPOSITORY_TOKEN` memakai token yang dapat dispatch ke repo mobile, atau memperbarui PAT lokal dengan izin Actions write agar secret dapat disimpan terenkripsi. Tanpa secret, CI sengaja gagal bila kontrak API berubah; CI perubahan non-kontrak tetap berjalan. Staging Render ditunda sampai seluruh alur local/CI lancar. Kredensial Google Maps kosong, tetapi bukan blocker Fase 0.
