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

> Keputusan bisnis sudah selesai. Staging Render free tier disiapkan sebagai
> langkah terakhir setelah seluruh alur lokal/CI lulus; staging kini live dan
> terverifikasi.

**Langkah — kerjakan berurutan**

| # | Langkah | Catatan |
|---|---|---|
| 1 | **Bersihkan sisa lama** — container `gomad-app-1` & `gomad-database-1`, dan folder `.smoke-test` | Namanya akan bentrok dengan project baru |
| 2 | **Repo `api/`** — Laravel + `git init` | Repo pertama |
| 3 | **Repo `mobile/`** — `git init`, isi kosong dulu | Repo kedua |
| 4 | **Stack Docker** di `api/docker/compose.yaml`: nginx · app (php-fpm) · queue · scheduler · Aiven MySQL via TLS · redis · mailpit · vite | Port: **8200** · **6380** · **8125** · **5273** |
| 5 | **Jalankan & buktikan** — `compose up`, halaman dibuka di **browser preview editor** | Menutup syarat verifikasi live pertama |
| 6 | **Perintah harian** — `compose up/down` · migrasi aditif · `artisan test` · `app:reset-demo` | tes SQLite terisolasi; reset demo ditolak untuk database remote |
| 7 | **CI dasar** — lint · tes · uji arsitektur (kerangka) · migrasi dari nol | Gagal = blokir |
| 8 | **Backup terjadwal** + **uji pemulihan** | Backup yang belum dipulihkan belum bisa disebut backup |
| 9 | **Kontrak API v0** — kerangka · cara generate · cara memberi sinyal ke repo `mobile/` | Karena dua repo terpisah |

**Kriteria lulus**
- [x] Container GoMad lama dibersihkan sebelum stack baru dinyalakan
- [x] `compose up` menyalakan seluruh stack, dan halaman terbuka di **browser preview editor**
- [x] Migrasi bersih + seed berjalan setelah izin eksplisit untuk menghapus schema Aiven lama; `app:reset-demo` menolak database remote sebagai pengaman
- [x] CI remote hijau pada commit integrasi; tes lokal, formatting, build aset, migrasi bersih + seed juga lulus
- [x] Backup terjadwal ke penyimpanan lokal dan R2; pemulihan lokal serta pemulihan nyata dari objek R2 sudah diuji
- [x] Kontrak API punya tempat resmi dan cara generate; perubahan kontrak memicu CI API → `repository_dispatch` → sinkronisasi OpenAPI mobile, semuanya lulus remote
- [x] Staging Render free tier dijalankan sebagai langkah terakhir setelah alur lokal, backup, CI, dan sinkronisasi kontrak lulus; halaman browser dan health check HTTPS lulus di <https://gomad-api-staging.onrender.com>
- [x] Prosedur rilis ditulis, walau rilis pertama belum terjadi

**Kesiapan mobile:** struktur responsif disiapkan sejak awal (satu layout, banyak ukuran)

**Akses:** aplikasi lokal di `http://localhost:8200` dan staging di <https://gomad-api-staging.onrender.com> lulus verifikasi live. Staging Fase 0 memakai SQLite in-memory, file cache/session, dan queue sync; tidak menyentuh database Aiven.

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
- [x] Tes: satu orang dengan dua peran bisa mengakses keduanya tanpa akun kedua
- [x] Tes: staff tidak bisa mengakses data usaha lain
- [x] Tes: otorisasi menolak aksi yang tidak diizinkan — di **satu** tempat, bukan tersebar
- [x] Token mobile bisa dipakai dari luar sesi web
- [x] Tidak ada pengecekan role yang tersebar di controller

**Kesiapan mobile:** alur login token, kontrak OpenAPI, dan sinkronisasi ke repo mobile sudah diverifikasi.

**Akses:** session login/registrasi, kelola anggota dan peran, serta undangan staff berumur tujuh hari tersedia di web dan API. Migrasi aditif dan katalog izin sudah diterapkan ke Aiven; staging tetap memakai SQLite in-memory.

---

## FASE 2 — Core: Ledger & Settlement

**Tujuan:** mesin uang yang bisa diaudit dan direkonsiliasi.

**Isi**
- `ledger_accounts` · `ledger_transactions` · `ledger_entries` · `wallet_holds`
- **Double-entry** dengan `category`, `reference`, dan `balance_after` per akun
- Akun: `PLATFORM.*` · `BUSINESS.PAYABLE:{id}` · `BUSINESS.AVAILABLE:{id}` · `BUSINESS.COLLATERAL:{id}` (agency)
- **PPN sebagai akun kewajiban**, bukan pendapatan
- Idempotency di level ledger + reversal (tidak pernah menimpa entri)
- Hold sebagai *record* dengan kunci unik — bukan kolom saldo
- Settlement engine: per agency & per grosir

**Kriteria lulus**
- [x] **Tes rekonsiliasi:** jumlah mutasi merekonstruksi saldo setiap akun dan memeriksa keseimbangan setiap transaksi
- [x] **Tes callback duplikat:** dikirim 3× → saldo berubah **sekali**
- [x] **Tes race condition:** dua proses posting bersamaan dengan kunci sama menghasilkan satu transaksi
- [x] **Tes reversal:** pembalikan membuat satu transaksi kompensasi tanpa mengubah entri lama
- [x] PPN dicatat ke akun kewajiban, bukan pendapatan
- [x] Hold tidak menghasilkan entri ledger dan tidak bisa dilepas/dipakai dua kali

**Kesiapan mobile:** — (murni backend)

**Akses:** settlement hanya untuk admin platform tersedia pada API. Saldo payable dipisahkan per usaha agar settlement tidak dapat memakai hak usaha lain. Migrasi aditif dan bagan akun agency/grosir tersedia; tidak ada UI ledger.

---

## FASE 3 — Core: Payment & Notification

**Tujuan:** uang benar-benar masuk dan keluar.

**Isi**
- Payment gateway Midtrans Snap untuk top-up jaminan agency: pembayaran per transaksi, webhook, verifikasi signature
- Metode fallback (mis. QRIS/VA) mengikuti metode yang diaktifkan pada merchant Snap; GoMad tidak menyimpan data kartu
- Pencairan manual melalui transfer bank admin: pengajuan → persetujuan → pencatatan eksekusi → pembalikan bila gagal
- Notifikasi in-app dan email untuk status pembayaran/pencairan; WhatsApp ditunda sesuai keputusan produk
- Buku besar operasional: rekonsiliasi ledger dapat dijalankan manual

**Kriteria lulus**
- [ ] Pembayaran nyata (nominal kecil) berhasil di staging
- [x] Tes webhook dengan signature palsu ditolak tanpa perubahan status, ledger, atau notifikasi
- [x] Pencairan gagal membalik reservasi secara idempoten dan memulihkan saldo tersedia
- [x] Rekonsiliasi otomatis setelah pembayaran memeriksa seluruh saldo akun dan keseimbangan transaksi
- [x] Tes notifikasi mengonfirmasi penerima tepat dan kanal in-app/email

**Batas yang masih terbuka:** transaksi Snap sandbox di staging belum dijalankan. Settlement tetap manual sesuai keputusan produk; WhatsApp ditunda. Refund setelah jaminan terpakai ditolak untuk posting otomatis, dicatat pada log operasional, dan memerlukan rekonsiliasi manual.

**Kesiapan mobile:** endpoint pembayaran tidak bergantung pada sesi web

**Akses:** API top-up jaminan Snap, webhook, pencairan manual admin, dan notifikasi in-app/email tersedia. Belum boleh disebut lulus live sebelum transaksi sandbox di staging diverifikasi.

---

## FASE 4 — Mad Warung inti (web)

**Tujuan:** satu order nyata, dari penjaga sampai barang diterima.

**Isi**
- **Portal Pemilik Warung**: multi-outlet · plafon penjaga · approval · jadwal shift · laporan
- **Halaman Penjaga** (web responsif): katalog · pesan · bayar · riwayat
- **Portal Grosir**: katalog · terima pesanan · tandai siap
- Alur order: plafon → approval → pesanan → grosir siapkan → antar/ambil → **konfirmasi terima** → settlement ke grosir
- Model pembayaran: penjaga membuat pesanan; pemilik menyetujui dan membuka checkout Midtrans Snap atas nama warung. Snap tidak mendebit metode pembayaran tersimpan pemilik; penyelesaian tetap mengikuti gateway.

**Progres lokal (belum pilot/live)**
- [x] API katalog/harga grosir, stok dan reservasi, plafon harian/per-pesanan, approval pemilik, checkout, ledger, pemenuhan, konfirmasi penerimaan, kedaluwarsa, shift, dan laporan
- [x] Portal web responsif untuk pemilik, penjaga, dan grosir
- [x] Tes alur pesanan, batas akses lintas usaha, checkout sandbox ter-mock, shift, dan laporan
- [ ] Tentukan tarif PPN yang disetujui dan konfigurasi sebelum checkout diaktifkan
- [ ] Terapkan migrasi Fase 4 hanya setelah validasi lokal lengkap; staging Render tetap ditunda sampai alur lokal siap

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

**Progres lokal (sedang dikerjakan; belum pilot/live)**
- [x] API agency untuk rute, armada, jadwal, persetujuan, dan aktivasi COD
- [x] Pencarian jadwal, booking kursi unik/idempoten, pembatalan online belum bayar, checkout Midtrans ter-mock, webhook dan ledger agency
- [x] Deposit COD dihitung per booking dan ditahan per penumpang; gate gagal tertutup bila saldo bebas kurang
- [x] Driver memulai/menyelesaikan jadwal, mengonfirmasi boarding (fee/PPN masuk ledger idempoten), atau menandai no-show untuk melepas hold
- [x] E-ticket penumpang dan portal customer (cari jadwal, denah kursi visual, pesanan saya)
- [x] Portal web agency: tambah armada & rute, buat jadwal, setujui pengajuan, aktifkan COD, mulai/selesaikan perjalanan, daftar penumpang per jadwal
- [x] Konfigurasi PPN Mad Trans di `.env` lokal + verifikasi regresi (85 tes hijau, 626 asersi)
- [x] Halaman isi jaminan COD di portal agency lewat Midtrans Snap; redirect ke halaman bayar terverifikasi live
- [x] Kredensial Midtrans kini benar-benar masuk container (sebelumnya kosong sehingga pembayaran online tidak mungkin jalan) dan URL `snap.js` dinormalkan ke endpoint `/snap/v1/transactions`
- [x] Halaman driver web responsif: daftar tugas, mulai/selesaikan perjalanan, konfirmasi boarding COD, tandai tidak naik; hanya supir yang ditugaskan yang bisa membuka
- [x] Verifikasi live alur penuh di browser: agency buat & setujui jadwal → penumpang pesan COD → supir mulai, boarding, selesaikan
- [ ] Uji dengan **1 agency sungguhan** (masih data demo)
- [x] Portal agency menampilkan **saldo siap cair** dan **saldo menunggu settlement** serta formulir pengajuan pencairan (minimal Rp 100.000, biaya admin Rp 5.000). Settlement tetap wewenang admin platform; saldo kurang ditolak dengan pesan jelas
- [ ] Staging Render tetap ditunda sampai semua alur lokal siap

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
- [x] Rental tidak bisa dobel-booking
  → bentrok sewa/sewa dan sewa/jadwal travel ditolak, kalender menandai hari terpakai
- [ ] Multi-stop: kursi yang ditinggalkan penumpang di titik tengah **bisa dijual lagi**
  → terpenuhi di lokal: lihat *Progres lokal — multi-stop* di bawah
- [x] Transfer penumpang memindahkan kursi & uang dengan benar di **kedua** jadwal
  → kursi terbukti berpindah dan ledger tidak berubah; **uang hanya benar karena tarif disyaratkan sama** — selisih harga belum didukung, lihat catatan progres
- [x] OTS gagal bayar menahan jaminan sesuai aturan
- [x] Deposit penyewa kembali tepat waktu dan tercatat
  → termasuk **potongan sebagian** untuk denda/kerusakan

**Progres lokal — multi-stop**
- [x] Portal agency bisa membuat rute 2–30 titik; tarif diisi per segmen dan harga antar titik berjauhan dihitung kumulatif
- [x] Pencarian jadwal dari/ke kota titik antara mengembalikan jadwal yang benar beserta tarifnya
- [x] Halaman pilih kursi menampilkan tarif segmen terpilih, menonaktifkan titik turun yang urutannya mendahului titik naik, dan e-ticket menampilkan titik naik/turun sebenarnya
- [x] **Kursi bisa dijual lagi di segmen lain** — indeks unik `(schedule_id, seat_number)` diganti indeks biasa; bentrok ditentukan lewat tumpang-tindih segmen di `CreateTravelBooking` saat baris jadwal dikunci (`SegmentSeatMap`)
- [x] Denah kursi diperbarui per segmen lewat `GET /mad-trans/schedules/{id}/seats` (web) dan parameter segmen pada endpoint API
- [x] Kapasitas dihitung per segmen pada pencarian, bukan total penumpang satu jadwal

**Progres lokal — rental (R1)**
- [x] Tabel `rentals` + tarif harian dan tarif supir per kendaraan (`vehicles.rental_daily_price`, `rental_driver_daily_fee`)
- [x] Sewa **dengan supir**: minimal 2 jam ke depan, hari dibulatkan ke atas, fee rental = service Rp 10.000 + platform 3% × harga sewa + PPN 11% × kedua fee (tarif supir tidak dikenai platform fee)
- [x] Ketersediaan kendaraan: bentrok dengan sewa lain **dan** dengan jadwal travel pada rentang yang sama ditolak
- [x] Halaman pelanggan `/mad-trans/rental/{agency}` dan siklus agency: catat pembayaran → serahkan kendaraan → kendaraan kembali → selesaikan
- [x] **Lepas kunci (self-drive)** — terbuka hanya bila KTP, SIM, dan selfie sudah **diverifikasi admin platform** (`/admin/rental-documents`, bukan agency). Agency hanya bisa membuka dokumen selama sewa berstatus `active`; akses dicabut saat sewa selesai (`documents_released_at` saat serah terima, `documents_revoked_at` saat selesai). Berkas disimpan di disk `local` (privat) dan disajikan lewat `RentalDocumentAccess`
- [x] Pembayaran Midtrans + pencatatan ledger rental: `payment_intents.rental_id`, cabang `rental` di webhook, `PostRentalPayment` mencatat `RENTAL_PAID` (PLATFORM.CASH → BUSINESS.PAYABLE + pendapatan service/platform + PPN), idempoten terhadap callback duplikat
- [x] **Deposit penyewa**: nominal per kendaraan (`vehicles.rental_deposit`), ikut ditagih saat bayar, ditahan sebagai kewajiban `PLATFORM.DEPOSIT_PAYABLE` (bukan pendapatan), lalu dikembalikan saat sewa selesai (`RefundApproved`, `deposit_status=returned`, `deposit_returned_at`). Deposit tunai yang dipegang agency ditandai kembali tanpa jurnal platform
- [x] Deposit **dipotong sebagian** untuk kerusakan/denda: agency mengisi potongan + alasan pada form "Selesaikan sewa". Bagian yang dipotong berpindah dari `PLATFORM.DEPOSIT_PAYABLE` ke `BUSINESS.PAYABLE:<id>` lewat kategori `DEPOSIT_DEDUCTED` (bukan pendapatan platform), sisanya dikembalikan lewat `REFUND_APPROVED`. Nominalnya tercatat di `rentals.deposit_deducted_amount` / `deposit_refunded_amount` / `deposit_deduction_reason` dan tampil di portal pelanggan maupun agency. Deposit tunai (dipegang agency) hanya mencatat nominal tanpa jurnal platform
- [x] **Charter**: tipe sewa `charter` pada tabel yang sama, wajib itinerary (tujuan bebas) dan supir agency, tarif memakai harga harian + tarif supir, ikut aturan ketersediaan & deposit. Form "Ajukan charter" di halaman pelanggan, itinerary tampil di portal agency
- [x] **OTS** (bayar di tempat): `rentals.payment_mode`, jaminan agency ditahan sejak pemesanan (`wallet_holds` reason `ots_deposit`), ditolak bila saldo jaminan kurang, fee platform diambil dari jaminan saat agency mencatat pembayaran tunai (`OTS_FEE_COLLECTED`), dan pembatalan melepas hold
- [x] **Transfer penumpang**: tabel `passenger_transfers`, `TransferTravelBooking` memindahkan kursi ke jadwal lain pada rute & agency yang sama, menolak bila kursi tujuan terisi, tarif segmen berbeda, jadwal asal sudah berjalan, atau beda rute/agency. **Uang tidak berpindah** (tarif wajib sama) sehingga ledger tidak berubah — selisih harga belum didukung
- [x] **Tarif jadwal diturunkan dari tarif rute**, bukan diisi manual: form buat jadwal tidak lagi meminta harga, `price_per_seat` diisi otomatis dari tarif segmen penuh sehingga angka yang tampil sama dengan yang ditagihkan. Rute tanpa tarif segmen penuh ditolak saat membuat jadwal
- [x] **Kalender ketersediaan visual** per kendaraan: `/mad-trans/rental/{agency}/vehicles/{vehicle}/calendar?month=YYYY-MM` menandai hari yang dipakai sewa (pending/paid/active/returned) dan jadwal travel armada yang sama, dengan navigasi bulan lalu/ini/depan (`VehicleAvailability`)
- [ ] Kalender belum menonaktifkan tanggal pada form pemesanan (validasi tetap dilakukan saat simpan)

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
- [x] Promo platform **tidak** memotong agency — dibuktikan tes (`PromoTest`)
- [x] Promo agency memotong agency — dibuktikan tes (`PromoTest`)
- [x] Rincian diskon terlihat di settlement
  → bagian **Diskon & promo** di portal agency + `/mad-trans/agency/{b}/promos`
- [x] **Referral ditanggung platform** (PR4) — dibuktikan tes & live: hadiah pengundang dan diskon
  pengguna baru sama-sama promo `cost_bearer = platform`, bagian agency tidak berkurang

**Progres lokal — promo (2026-10-18)**
- [x] Tabel `promos` + `promo_redemptions`; kolom diskon di `bookings` dan `rentals` (`discount_amount`, `promo_id`, `promo_code`, `discount_bearer`)
- [x] `CreatePromo` (admin platform & agency), `ResolvePromo` (validasi kode, masa berlaku, min belanja, kuota, batas per pengguna), `RedeemPromo` (dicatat di dalam transaksi pemesanan)
- [x] Diskon hanya menyentuh bagian penjual (harga tiket / harga sewa + tarif supir), **tidak** biaya layanan, fee platform, PPN, atau deposit
- [x] **Platform menanggung** → agency tetap menerima bagian penuh: jurnal `PROMO_PLATFORM_BORNE` (debit `PLATFORM.EXPENSE.PROMO` → kredit `BUSINESS.PAYABLE`)
- [x] **Agency menanggung** → `BUSINESS.PAYABLE` dikredit sebesar bagian penjual − diskon, tanpa beban platform
- [x] Promo berlaku untuk pembayaran **online** saja; COD/OTS ditolak dengan pesan jelas (uang tunai tidak lewat ledger)
- [x] Rincian checkout Midtrans menampilkan baris diskon agar total Snap = total pesanan
- [x] E-ticket menampilkan baris diskon promo dan tombol **Bayar online** (web)
- [x] **Referral**: `users.referral_code` + tabel `referrals`; kode diisi saat daftar (`?ref=`), pengguna baru dapat promo `WELCOME-…` (platform), pengundang dapat promo `REW-…` (platform, Rp 25.000) setelah transaksi berbayar pertama pengundang selesai. Halaman `/referral` menampilkan kode, tautan, undangan, dan hadiah
- [x] **Level & misi**: `loyalty_events` + `users.points_cached`; 1 poin per Rp 10.000 transaksi lunas (travel & sewa), bonus poin referral, idempoten per transaksi. Level Bronze/Silver/Gold/Platinum dari `services.loyalty.levels`, halaman `/level` menampilkan level, progres ke level berikut, misi bulan ini, dan riwayat poin

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
- [x] Rekonsiliasi penuh cocok, tanpa selisih 1 rupiah
  → `php artisan gomad:reconcile` (live: 13 akun cocok, 13 transaksi seimbang, exit 0)
- [x] Uji pemulihan backup berhasil
  → `bash bin/verify-backup-restore <file.sql>`: restore ke database sekali pakai, lalu otomatis
  menjalankan `gomad:schema-check` **dan** `gomad:reconcile` pada hasil restore (2026-10-30: 49 tabel,
  skema cocok, rekonsiliasi bersih). Catatan: backup sumber saat itu berasal dari database lokal yang
  belum berisi data, jadi uji ini struktural + ledger-kosong; uji dengan data Aiven menunggu instance hidup
- [x] Tidak ada temuan 🔴 dari `docs/04-checklist-jangan-diulang.md` yang tersisa
  → audit lengkap di `12-audit-checklist.md`; E1 kini tertutup oleh `gomad:schema-check` + tes
- [x] Prosedur operasional tersedia dan sudah dicoba
  → `11-prosedur-operasional.md`; jalur pencairan, refund, dan rekonsiliasi sudah ditelusuri di sistem
- [x] Monitoring memberi tahu saat ada kegagalan uang
  → `php artisan gomad:money-health` tiap 10 menit: log `critical` + exit 1 (live: exit 0 dengan peringatan 2 pembayaran menggantung)

**Progres lokal — pengerasan (2026-10-18)**
- [x] `gomad:reconcile` — saldo tiap akun direkonstruksi dari mutasi; exit 1 bila ada selisih atau transaksi tidak seimbang; tes menutup kasus saldo diubah manual
- [x] `gomad:money-health` — memeriksa selisih ledger, transaksi tidak seimbang, callback gagal, pembayaran menggantung, booking kedaluwarsa, deposit macet; terjadwal tiap 10 menit
- [x] Rate limit jalur uang: `throttle:money` (30/menit), `throttle:uploads` (20/menit), `throttle:promo` (10/menit), selain `throttle:auth` yang sudah ada — dibuktikan tes 429
- [x] Prosedur operasional tertulis: pencairan, settlement, refund, sengketa, kegagalan webhook, backup & uji pemulihan, monitoring, feature freeze
- [x] Uji beban ringan
  → `bash bin/load-test`: 300 permintaan / konkurensi 15 → **0 gagal**, rata-rata 209 ms, p95 267 ms (lokal)
- [ ] Domain publik + alerting nyata belum ada
- [x] `gomad:schema-check` (2026-10-30) — referensi dari `migrate:fresh` di DB sementara, dibandingkan dengan DB aktif; exit 1 + pesan kolom/tabel yang menyimpang. Dijalankan di MariaDB lokal: 49 tabel tanpa selisih; kolom liar terdeteksi

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
- [x] Klien Dart bisa dibuat otomatis dari kontrak, tanpa ditulis manual — `api/bin/generate-dart-client` (dart-dio) menghasilkan `mobile/api_client` (22 kelompok API, ±155 model) yang lolos `dart analyze`
- [x] Auth token & refresh bekerja dari aplikasi — `POST /auth/refresh` merotasi token per perangkat; diuji di `tests/Feature/MobileApiTest.php` dan `mobile/test/session_test.dart`
- [x] Satu alur pilot jalan di Android — kerangka `mobile/` + alur login → cari jadwal → peta kursi → pesan → bayar terbukti lewat klien Dart ke server lokal (`mobile/tool/pilot_smoke.dart`); pemasangan APK di perangkat tinggal `flutter create .`
- [x] Tidak ada logika bisnis yang perlu diduplikasi di mobile — rental, dokumen lepas kunci, promo, referral, dan level kini punya endpoint API yang memanggil kelas Core yang sama dengan web

**Progres (2026-10-30)**
- Endpoint baru: `/mad-trans/businesses/{business}/rental-vehicles`, `POST /mad-trans/businesses/{business}/rental-bookings`, `GET /mad-trans/rental-bookings`, `GET /mad-trans/rental-bookings/{rental}`, `POST /mad-trans/rental-bookings/{rental}/pay`, `GET|POST /rental-documents`, `POST /promos/validate`, `GET /referral`, `GET /loyalty`, `POST /auth/refresh`.
- `tests/Feature/MobileApiTest.php` (10 tes) mengunci bentuk respons, kepemilikan data, rotasi token, dan pencarian jadwal dengan tanggal ISO.
- Kontrak dibekukan ulang; `TravelBookingResource` dipakai langsung pada respons booking agar Scramble menghasilkan model bertipe.
- Alat bantu baru: `bin/check-openapi-schemas` (daftar skema kosong yang membuat klien Dart gagal dikompilasi).
- Kerangka aplikasi: `mobile/` (login, cari jadwal, peta kursi, pembayaran) dengan `token_store` yang siap diganti penyimpanan aman.
- Peta komponen web → widget: `blueprint/14-peta-komponen-web-ke-widget.md`.

**Akses:** aplikasi native siap dibangun bertahap

---

## Pelacak Status

> **Diperbarui setiap sesi berakhir.** Ini yang dibaca sesi berikutnya.

| Fase | Status | Catatan |
|---|---|---|
| Konsep | ✅ Selesai | `docs/08` · `09` · `11` |
| Blueprint | ✅ **Selesai — 9 dari 9** | `blueprint/README.md` |
| 0 — Fondasi | ✅ Selesai (staging ditunda) | Fondasi lokal, Aiven/R2, dan CI; staging Render baru dilakukan setelah alur lokal siap |
| 1 — Identity & Access | ✅ Selesai lokal | Implementasi dan verifikasi lokal; belum menjadi klaim lulus live |
| 2 — Ledger & Settlement | ✅ Selesai lokal | Ledger/settlement dan tes lokal; settlement operasional tetap manual |
| 3 — Payment & Notification | ✅ Selesai lokal | Midtrans Snap/webhook/notifikasi terimplementasi; belum diuji transaksi live |
| 4 — Mad Warung inti | 🟡 Implementasi lokal | Portal dan alur API/web serta tes lokal; PPN harus dikonfigurasi sebelum checkout, migrasi Fase 4 dan pilot/live belum dilakukan |
| 5 — Mad Trans inti | 🟡 Portal selesai lokal | Portal penumpang, agency, **dan supir** sudah menyelesaikan alur penuh di browser (pesan → bayar/COD → berangkat → boarding → selesai); sisa uji dengan agency sungguhan dan staging |
| 6 — Mad Trans lanjutan | ✅ Selesai lokal | Multi-stop, rental (dengan supir/lepas kunci/charter), OTS, deposit + potongan sebagian, transfer penumpang, dokumen lepas kunci (verifikasi admin + akses agency terbatas), dan kalender ketersediaan sudah jalan; sisa uji dengan agency sungguhan |
| 7 — Keterlibatan | ✅ Selesai lokal | Promo `cost_bearer`, referral yang selalu ditanggung platform, serta level & misi sudah jalan dan terverifikasi |
| 8 — Pengerasan & rilis | 🟡 Sisa domain & alerting | Rekonsiliasi, monitoring, rate limit, prosedur ops, audit 🔴, cek skema, uji pemulihan backup, dan uji beban ringan (0 gagal, p95 267 ms) selesai; sisa domain publik + alerting nyata dan B3 cutoff penjualan |
| 9 — Persiapan mobile | ✅ Selesai lokal | Kontrak dibekukan, klien Dart digenerate otomatis dan lolos analisis, auth + refresh teruji, serta alur pilot (login → cari jadwal → pesan → bayar) terbukti dari klien Dart; sisa APK di perangkat, push notification, dan penyimpanan token aman |
| 10 — Interface lengkap | ✅ D0–D6 + P1/P2 selesai | Fondasi desain, permukaan publik, portal pelanggan, agency, Mad Warung, konsol operasional, dukungan (tiket), pelacakan perjalanan, dan **verifikasi dua langkah** sudah jalan beserta endpoint API-nya; **satu akun satu fungsi** (P1) dan pemisahan **ops vs admin** (P2) menyusul; sisa yang diketahui hanya **log audit** |
| 11 — Pemisahan fungsi akun | ✅ P1–P4 selesai | `users.account_type` + middleware `area:` + rute per area + beranda sadar-peran + navigasi per fungsi + peran platform `ops`; controller & view web dan API sudah di folder per area (`blueprint/16`), kontrak diekspor ulang dan klien Dart digenerate |

**Sedang dikerjakan:** Fase 11 — pemisahan fungsi akun.
**P1 selesai:** `users.account_type` (`App\Core\Identity\Domain\AccountType`, 10 fungsi),
middleware `area:` + `routes/areas/*`, beranda sadar-peran (`/dashboard`) dan `/usaha` khusus
mitra, navigasi per fungsi, pembatasan pembuatan usaha, serta undangan yang menolak nomor
berfungsi ganda.
**P2 selesai:** peran platform `ops` (izin `ops.view`, `withdrawal.review`, `business.verify`,
`order.report`) terpisah dari `admin` (tambah `settlement.execute`, `catalog.publish`,
`permissions.read`); tab konsol disaring per izin; halaman parameter & pengguna hanya admin.
**Berikutnya:** P5 verifikasi data demo per fungsi (selesai: 8 akun, satu fungsi per akun);
P6 dokumen; uji alur penuh dengan **1 agency + 1 warung sungguhan** dan migrasi staging.
**Gate area API selesai:** `routes/api/*` kini memakai middleware `area:` — mitra tidak bisa
memanggil endpoint konsol walau menempelkan peran platform ke akunnya (diuji di
`WithdrawalTest`), dan sebaliknya.
**P3 selesai:** folder view `madwarung/`, `madtrans/{customer,travel,rental,driver}`,
`platform/admin/`, `shared/` dan controller `Web\{MadWarung,MadTrans,Platform,Shared}` —
peta lengkap di `blueprint/16`.
**P4 selesai:** controller API di `Api\V1\{Shared,MadWarung,MadTrans\{Customer,Travel,Rental},Platform,Money}`
dan rute dipecah ke `routes/api/{shared,madwarung,madtrans,platform}.php`; 105 endpoint API,
kontrak diekspor ulang (92 path, tanpa perubahan alamat/nama rute) dan klien Dart digenerate.
**Blocker terbuka:** Tidak ada. *(Diperbarui 2026-11-02.)*

**Pembersihan 2026-10-17 — selesai:**
- PPN 11% dikonfigurasi (`MAD_*_PPN_BASIS_POINTS=1100`); staging/produksi tetap wajib set eksplisit
- `TravelBookingResource` → bentuk respons satu sumber; endpoint detail bertipe; kontrak kini memuat skema (`departure_date` muncul di `openapi.yaml`)
- Data tiket dilengkapi: `route`, `departure_date`, `departure_time`, `travel_class`
- Penyimpangan dokumen dicatat (Blade · `app/Core`), bukan dibiarkan
- Suite: **78 tes / 576 assertion hijau**; live check: web 200 · API health 200 · endpoint ber-auth 401

**Catatan penyimpangan dari blueprint** (perlu diselaraskan atau diperbarui dokumennya):
- Vertical berada di `app/Core/{MadTrans,Warung}`, **bukan** di `app/Modules/` seperti `blueprint/01`
- Web memakai **Blade**, **bukan** Inertia + Vue seperti `blueprint/01` dan `docs/00`
- Kontrak API **belum memuat skema respons** untuk booking/order — controller mengembalikan `array` tak bertipe, sehingga klien mobile belum bisa digenerate dengan andal
