# Blueprint 04 — Layar per Peran

> Satu wadah web, banyak peran (keputusan: web dulu untuk semua peran).
> Rujukan: `blueprint/01` (pola peran) · `03` (kontrak API) · `05` (aturan bisnis)

---

## 1. Prinsip

| # | Prinsip |
|---|---|
| 1 | **Satu aplikasi web, banyak peran.** Menu & izin berbeda, kode sama |
| 2 | **Mobile-first untuk Staff.** Penjaga & Driver memakai web di HP — desain dari layar kecil ke besar, bukan sebaliknya |
| 3 | **Desktop-first untuk Pemilik & Admin.** Mereka bekerja dengan tabel, laporan, dan banyak baris |
| 4 | **Setiap halaman punya 4 keadaan:** memuat · kosong · galat · terisi. Yang kosong & galat dirancang, bukan dibiarkan |
| 5 | **Tidak ada halaman tanpa jalur keluar.** Setiap keadaan buntu harus punya tindakan berikutnya |
| 6 | **Satu akun satu fungsi.** Fungsi akun menentukan area yang boleh dibuka; merangkap fungsi = membuat akun lain |

---

## 1a. Fungsi akun — sumber kebenaran akses

Fungsi akun disimpan di `users.account_type` (`App\Core\Identity\Domain\AccountType`) dan
menentukan **area** yang boleh dibuka. Rute dijaga middleware `area:` dengan daftar area
yang sama, jadi menu dan penjagaan tidak bisa berbeda pendapat.

| Fungsi akun | Label | Area | Beranda setelah masuk |
|---|---|---|---|
| `customer` | Pelanggan | `customer` | `/dashboard` (beranda belanja) |
| `warung_owner` | Pemilik warung | `retail` | `/mad-warung` |
| `warung_guard` | Penjaga warung | `retail` | `/mad-warung` |
| `grosir_owner` | Pemilik grosir | `grosir` | `/mad-warung` |
| `grosir_staff` | Staf grosir | `grosir` | `/mad-warung` |
| `agency_owner` | Pemilik travel & rental | `agency` | `/mad-trans/agency` |
| `agency_staff` | Staf travel & rental | `agency` | `/mad-trans/agency` |
| `driver` | Supir | `driver` | `/mad-trans/driver` |
| `ops` | Ops platform | `ops` | `/admin` |
| `admin` | Admin platform | `ops` + `admin` | `/admin` |

**Aturan yang menyertainya**

| # | Aturan |
|---|---|
| 1 | Akun mitra memilih fungsinya saat **pendaftaran** (`customer`, `warung_owner`, `grosir_owner`, `agency_owner`); staf masuk lewat **undangan** usaha |
| 2 | Pelanggan yang ingin menjadi mitra **mendaftar akun baru** — akun tidak berganti fungsi |
| 3 | Undangan staf ditolak bila nomor tujuan sudah dipakai fungsi lain |
| 4 | Akun pemilik hanya bisa membuat usaha **sesuai fungsinya** (warung → warung, grosir → grosir, agency → agency) |
| 5 | Area `shared` (profil, notifikasi, dukungan, referral, level, anggota usaha) terbuka untuk semua akun |

**Daftar area dan rutenya**

| Area | Rute | Berkas rute |
|---|---|---|
| `retail`, `grosir` | `/mad-warung/**` | `routes/areas/madwarung.php` |
| `agency` | `/mad-trans/agency/**` | `routes/areas/madtrans-agency.php` |
| `driver` | `/mad-trans/driver/**` | `routes/areas/madtrans-driver.php` |
| `customer` | `/mad-trans`, `/mad-trans/bookings/**`, `/mad-trans/schedules/**`, `/mad-trans/rental/**` | `routes/areas/madtrans-customer.php` |
| `ops` (ops & admin) | `/admin/**` | `routes/areas/platform.php` |
| bersama | `/dashboard`, `/usaha`*, `/profil`, `/notifikasi`, `/dukungan`, `/businesses/**` | `routes/web.php` |

\* `/usaha` khusus akun mitra (`area:retail,grosir,agency`), karena hanya mereka punya usaha.

**Beranda** (`/dashboard`) meneruskan akun ke ruang kerjanya sendiri; hanya pelanggan yang
melihat halaman (beranda belanja). URL tetap satu supaya tautan "beranda" bisa dipakai
di mana pun.

**API juga dijaga, bukan cuma web.** Grup rute `/api/v1` memakai middleware `area:` yang sama:
`/warung/**` → `area:retail,grosir`; `/mad-trans/**` → `area:customer`, `area:driver`, atau
`area:agency` sesuai bagiannya (mulai/tutup perjalanan boleh agency **dan** supir);
`/admin/**` → `area:ops`. Akun mitra yang memanggil endpoint konsol mendapat `403` dengan
kode `FORBIDDEN` sebelum izin per aksi diperiksa.

---

## 1b. Navigasi per fungsi akun

Header hanya memuat **pintu masuk** ruang kerja, bukan seluruh isi ruang kerja — daftar
halaman per pekerjaan ada di sub-navigasi ruang kerja (`_nav.blade.php`), disaring dengan
izin yang sama.

| Fungsi akun | Menu header |
|---|---|
| Pelanggan | Beranda · Cari jadwal · Pesanan saya · Sewa saya · Mad Warung · Level & misi |
| Pemilik/penjaga warung | Ruang kerja warung · Usaha saya |
| Pemilik/staf grosir | Ruang kerja grosir · Usaha saya |
| Pemilik/staf agency | Ruang kerja agency · Usaha saya |
| Supir | Jadwal saya |
| Ops | Konsol · Pencairan · Mitra · Rekonsiliasi |
| Admin | semua menu ops + Settlement · Pengguna & peran · Parameter |

**Pemisahan ops vs admin.** Konsol platform satu pintu, dua tingkat pekerjaan:

| Halaman | Izin | Ops | Admin |
|---|---|---|---|
| Dasbor · Rekonsiliasi · Dukungan · Dokumen sewa | `ops.view` | ✅ | ✅ |
| Pencairan | `withdrawal.review` | ✅ | ✅ |
| Verifikasi mitra | `business.verify` | ✅ | ✅ |
| Settlement | `settlement.execute` | ❌ | ✅ |
| Promo platform | `catalog.publish` | ❌ | ✅ |
| Pengguna & peran · Parameter | `permissions.read` | ❌ | ✅ |

---

## 2. Pihak Luar — Customer

📱 **Mobile-first** · Fase 4 (travel) & 6 (rental)

| # | Halaman | Isi penting | Status |
|---|---|---|---|
| 1 | Cari jadwal | Rute · tanggal · jumlah penumpang | ✅ `/mad-trans` & publik `/jadwal` |
| 2 | Hasil pencarian | Bandingkan jadwal antar agency · harga per segmen transparan | ✅ |
| 3 | Pilih kursi | Denah kursi · kursi supir terkunci · kursi terisi tampak jelas | ✅ `/mad-trans/schedules/{id}` |
| 4 | Checkout | Alamat jemput & antar · data penumpang · promo | ✅ (form pada halaman pilih kursi) |
| 5 | Pembayaran | Metode utama + **fallback QRIS/VA** bila auto-charge gagal | 🟡 Snap redirect; fallback di dalam halaman Midtrans |
| 6 | E-ticket | Kode, kursi, detail, bisa ditunjukkan ke supir | ✅ `/mad-trans/bookings/{id}` |
| 7 | Pesanan saya | Daftar + detail + status | ✅ `/mad-trans/bookings` |
| 8 | Pembatalan | Estimasi refund sebelum konfirmasi — **bukan setelah** | ✅ `/mad-trans/bookings/{id}/batalkan` (hanya bila belum ada dana ditarik) |
| 9 | Rental | Cari kendaraan · kalender ketersediaan · unggah KTP/SIM/selfie | ✅ `/mad-trans/rental/{business}` · `/mad-trans/rental-documents` · riwayat di `/profil/sewa` |
| 10 | Profil & notifikasi | | ✅ `/profil` · `/notifikasi` |

**Alur wajib berhasil:** cari → pilih kursi → bayar → e-ticket terbit.

**Catatan pembatalan:** sistem hanya membatalkan sendiri pesanan yang **belum menarik uang**
(pending online tanpa payment intent) dan COD yang belum berangkat. Pesanan yang sudah dibayar
menjadi urusan ops (refund) — aturannya satu, di `TravelBookingCancellationPolicy`, dipakai
layar konfirmasi, tombol batal, dan API sekaligus.

---

## 3. Staff — Penjaga Warung

📱 **Mobile-first** · Fase 5

| # | Halaman | Isi penting |
|---|---|---|
| 1 | Beranda | Status shift · ringkasan hari ini · pengingat jaminan |
| 2 | Katalog | Pilih grosir · cari produk · keranjang |
| 3 | Checkout | Rincian biaya · **sisa plafon terlihat jelas** · tombol bayar |
| 4 | Menunggu approval | Status permintaan · siapa yang harus menyetujui |
| 5 | Pembayaran | QRIS/VA bila auto-charge gagal |
| 6 | Pesanan saya | Daftar · detail · status pengiriman |
| 7 | Konfirmasi penerimaan | Tombol konfirmasi barang diterima |
| 8 | Shift saya | Jadwal & riwayat |

**Alur wajib berhasil:** pilih produk → bayar → konfirmasi terima.

**Keadaan yang wajib dirancang:** melewati plafon · pembayaran gagal · pesanan ditolak pemilik.

**Status D4:** penjaga memakai ruang kerja yang sama dengan pemilik, dengan tab terbatas —
`/mad-warung/{business}` (dasbor + shift aktif), `.../katalog` (pesan stok), `.../pesanan`
(hanya pesanannya sendiri), `.../shift` (mulai/akhiri shift miliknya). Halaman
`.../penjaga` dan `.../laporan` sengaja **ditolak** (403) untuk penjaga.

**Plafon terlihat sebelum mengetik (D7).** Halaman `.../katalog` kini menampilkan kartu
**"Sisa plafon hari ini"** — sisa, terpakai/dari, batang pemakaian, maksimal per pesanan, dan
ambang persetujuan. Angkanya dibaca dari `member_limits` + `orders` hari ini dengan query yang
**sama** dengan `CreateWarungOrder`, jadi tidak ada aturan kedua. Subtotal barang dihitung
langsung di browser; **biaya platform dan PPN tidak** — keduanya tetap milik server, karena
menyalin rumus uang ke JavaScript adalah cara termudah membuat dua angka berbeda untuk
pesanan yang sama.

### POS (kasir) — v1 selesai

UDS §7.4 memberi Penjaga Warung Retail daftar **Point of Sale**. Sebelumnya domain
itu belum ada (yang ada hanya `Order` — pesanan warung ke grosir, B2B). Sekarang
sudah ada sebagai fitur penuh, bukan tampilan kosong.

| Halaman | Rute | Siapa |
|---|---|---|
| Kasir | `GET/POST /mad-warung/{business}/kasir` | penjaga (`pos.sell`) |
| Penjualan | `GET /mad-warung/{business}/penjualan` | pemilik (`pos.report`) |

**Tiga keputusan uang yang dipegang** (semuanya disengaja, lihat `RecordRetailSale`):

| # | Keputusan | Alasan |
|---|---|---|
| 1 | **Tidak memposting ke ledger** | `BUSINESS.AVAILABLE:<id>` berarti "uang yang dipegang platform dan bisa dicairkan". Kasir menerima tunai di meja warung; memasukkannya ke sana akan membuat pemilik bisa mencairkan uang yang tidak pernah diterima platform. Penjualan tetap terekam di `retail_sales` dan bisa diaudit |
| 2 | **Tanpa biaya platform, tanpa PPN** | Platform bukan penjual pada transaksi ini dan tidak memproses pembayarannya. `total_amount` = jumlah baris, tanpa potongan tersembunyi |
| 3 | **Stok grosir tidak disentuh** | `product_prices.stock_available` adalah stok **grosir**, bukan rak warung. Persediaan warung belum dimodelkan; kasir tidak berpura-pura memilikinya |

**Cara barang masuk ke kasir.** Saat warung mengonfirmasi **penerimaan** pesanan
grosir, tiap produk otomatis terdaftar di `retail_listings` dengan **harga beli
terakhir** sebagai titik awal (`ConfirmOrderReceipt`). `firstOrCreate` menjaga harga
yang sudah disesuaikan pemilik tidak tertimpa saat barang yang sama dipesan lagi.
Penjaga tidak pernah menentukan harga.

**Kenapa pemilik hanya memantau.** Izin `pos.sell` hanya dipegang `penjaga`; pemilik
memegang `pos.report`. Ini bukan sekadar menyembunyikan menu — `RecordRetailSale`
memanggil `ActionAuthorizer` pada `pos.sell`, jadi pemilik yang memaksa membuka
`/kasir` akan ditolak 403.

**Yang belum ada di POS v1:** retur/pembatalan nota, diskon, pembayaran non-tunai,
dan persediaan warung. Masing-masing adalah fitur tersendiri, bukan tambalan layar.

---

## 4. Staff — Driver

📱 **Mobile-first** · Fase 4 · tema **Classic Minimalist** (UDS §7.4)

| # | Halaman | Isi penting |
|---|---|---|
| 1 | Tugas saya | Tugas aktif (jam 3xl + tanggal + status) dan blok terpisah **"Selesai — 7 hari terakhir"** |
| 2 | Detail jadwal | Status, jam berangkat, rasio penumpang naik/total, pelat kendaraan |
| 3 | Mulai / selesai perjalanan | Satu tombol utama selebar layar, tinggi 56px — berganti sesuai keadaan |
| 4 | Konfirmasi COD | Dua tombol besar per penumpang (48px): "Konfirmasi naik" / "Tidak naik" |
| 5 | Ajukan transfer penumpang | Bila ada penumpang yang perlu dialihkan |

**Aturan keselamatan yang dipegang layar supir:**

- **Satu tindakan utama per keadaan.** Saat belum berangkat hanya ada "Mulai
  perjalanan"; saat berjalan hanya ada "Selesaikan perjalanan". Tidak ada deretan
  tombol serupa yang harus dibaca sambil mengemudi.
- **Tidak ada tabel.** Daftar tugas berupa kartu satu kolom; seluruh kartu adalah
  satu tautan besar, jadi target sentuhnya jauh di atas 44px.
- **Status terbaca sekilas.** Badge status + jam berukuran 3xl; status tidak pernah
  hanya dibedakan warna (UDS §12.5).
- **Perjalanan selesai dipisah tegas** dari yang aktif, dibatasi 7 hari terakhir agar
  supir bisa memastikan perjalanan mana yang sudah ia tutup saat ada sengketa.

---

## 5. Pemilik Usaha — Pemilik Warung

💻 **Desktop-first** · Fase 5 · **D4 selesai** untuk pemilik warung (layar penjaga masih mobile-first)

| # | Halaman | Isi penting | Rute web |
|---|---|---|---|
| 1 | Dasbor | Ringkasan **semua outlet** · shift aktif · penjaga mendekati plafon | `/mad-warung/{business}` |
| 2 | Outlet | Tambah & kelola warung | `/mad-warung` · `/dashboard` |
| 3 | Penjaga | Undang · aktif/nonaktif · detail | `/businesses/{business}` |
| 4 | **Plafon & batas** | Batas harian · batas per pesanan · ambang approval · pemakaian hari ini | `.../penjaga` |
| 5 | **Jadwal shift** | Siapa bertugas kapan · penjaga mulai/akhiri shift sendiri | `.../shift` |
| 6 | **Approval** | Permintaan menunggu · setujui/tolak **+ alasan** | `.../penjaga` · `.../pesanan` |
| 7 | Pesanan | Per outlet · filter status · aksi sesuai peran | `.../pesanan` |
| 8 | Laporan | Per outlet **dan** gabungan · produk terlaris | `.../laporan` |
| 9 | Pesan stok | Katalog grosir · keranjang · metode pengiriman | `.../katalog` |

**Alur wajib berhasil:** approval dari jarak jauh saat penjaga melewati plafon.
Terbukti lokal: penjaga memesan → owner melihat "Menunggu persetujuan"/"Menunggu bayar"
sampai 30.999 → grosir melihat pesanan masuk → laporan kedua sisi menghitung pesanan yang sama.

---

## 6. Pemilik Usaha — Grosir

💻 **Desktop-first** · Fase 5 · **D4 selesai**

| # | Halaman | Isi penting | Rute web |
|---|---|---|---|
| 1 | Dasbor | Pesanan hari ini · perlu disiapkan | `/mad-warung/{business}` |
| 2 | Katalog & harga | Produk · harga **per grosir** · minimum order · stok | `.../katalog` |
| 3 | Pesanan masuk | Konfirmasi · tandai siap · tandai dikirim | `.../pesanan` |
| 4 | Riwayat & settlement | Pesanan per status · warung teratas · produk terlaris | `.../laporan` |

Satu controller (`WarungPortalController`) melayani warung dan grosir: tab berbeda,
hak akses berbeda, data yang sama diambil dari sudut pandang berbeda. Endpoint API
pasangannya sudah ada sejak Fase 5 (`/warung/orders`, `/warung/catalog`, `/warung/reports`,
`/warung/businesses/{business}/member-limits`, `/warung/businesses/{business}/shifts`).

---

## 7. Pemilik Usaha — Agency (travel & rental)

💻 **Desktop-first** · Fase 4 & 6 · **D3 selesai** (satu halaman per pekerjaan, bukan satu halaman raksasa)

| # | Halaman | Isi penting | Rute web |
|---|---|---|---|
| 1 | Dasbor | Ringkasan penjualan & jadwal · saldo · pintasan | `/mad-trans/agency/{business}` |
| 2 | Armada | Kendaraan · **denah kursi** · tarif sewa harian | `.../armada` |
| 3 | Rute | Titik pemberhentian · **harga per segmen** | `.../rute` |
| 4 | Jadwal | Buat · terbitkan · setujui · mulai · selesaikan | `.../jadwal` |
| 5 | Pengemudi | Tugaskan ke jadwal · anggota tim | `.../pengemudi` |
| 6 | Pesanan & penumpang | Daftar · filter status · daftar kursi · transfer | `.../pesanan` |
| 7 | Rental | Kendaraan · dokumen penyewa · serah terima | `.../rental` |
| 8 | **Dompet** | Saldo tersedia & jaminan · mutasi · hold aktif | `.../dompet` |
| 9 | Settlement & pencairan | Ajukan · riwayat | `.../pencairan` |
| 10 | Laporan | Harian · per rute · rincian settlement · promo | `.../laporan` |

Setiap halaman memakai `x-gm.page-header` + tab `_nav` yang sama, dan diakses lewat
satu pintu hak akses `schedule.manage` pada agency tersebut. Paritas API wajib:
`GET /api/v1/mad-trans/businesses/{business}/{wallet,bookings,withdrawals,rental-bookings}`.

**Alur wajib berhasil:** terbitkan jadwal → terima pesanan → dana masuk dompet → cairkan.

---

## 8. Admin / Ops

💻 **Desktop** · Fase 1–3 · **D5 selesai** (kecuali sengketa & log audit — belum ada domainnya)

| # | Halaman | Isi penting | Rute web |
|---|---|---|---|
| 1 | Dasbor operasional | Antrean yang menunggu keputusan · kesehatan ledger · peringatan konfigurasi | `/admin` |
| 2 | Verifikasi | Mitra (verifikasi/cabut) · dokumen KTP/SIM/selfie (halaman lama dipertahankan) | `/admin/mitra` · `/admin/rental-documents` |
| 3 | Pengguna & peran | Siapa punya peran platform apa · jumlah usaha | `/admin/pengguna` |
| 4 | **Keuangan** | Pencairan (setujui/bayar/gagalkan) · settlement · **rekonsiliasi** | `/admin/pencairan` · `/admin/settlement` · `/admin/rekonsiliasi` |
| 5 | Parameter platform | Fee · PPN · batas waktu · minimum pencairan (hanya baca) | `/admin/parameter` |
| 6 | Sengketa | Penanganan keluhan | ⏳ D6 (butuh domain tiket) |
| 7 | Log audit | Siapa melakukan apa | ⏳ belum ada tabel audit (tercatat sebagai sisa Fase 8) |

> **Halaman rekonsiliasi adalah yang terpenting di sini.** Kalau ops tidak bisa mencocokkan uang dengan cepat, setiap selisih jadi krisis.
> Rekonsiliasi menjalankan `ReconcileLedger` yang sama dengan tes ledger, jadi angka di layar = angka di tes.

**Aturan yang dijaga:** aksi uang di konsol memakai kelas Core yang sama dengan API
(`ApproveWithdrawal`, `CompleteWithdrawal`, `FailWithdrawal`, `SettleBusinessFunds`), dan
parameter platform **tidak bisa diubah dari layar** — perubahan harus lewat konfigurasi deploy
agar ada jejak di repositori.

---

## 9. Aturan responsif

| Peran | Titik desain | Alasan |
|---|---|---|
| Penjaga · Driver · Customer | **360px** dulu, baru melebar | Dipakai sambil berdiri, satu tangan |
| Pemilik Warung · Grosir | **1280px** dulu, tetap layak di 768px | Bekerja dengan tabel & laporan |
| Agency | **1280px** dulu | Banyak kolom |
| Admin | **1440px** dulu | Kepadatan data tinggi |

**Aturan:** halaman Staff **tidak boleh** mengharuskan menggulir ke samping. Tabel di halaman Pemilik boleh, asal ada versi kartu untuk layar kecil.

---

## 10. Komponen bersama

Dibuat sekali, dipakai semua peran — supaya tidak ada tiga cara menampilkan hal yang sama:

| Komponen | Dipakai di |
|---|---|
| Kartu ringkasan | Semua dasbor |
| Tabel + versi kartu | Pemilik · Grosir · Agency · Admin |
| Pemilih tanggal/waktu | Jadwal · shift · rental |
| Badge status | Semua (warna konsisten per status) |
| Rincian biaya (fee + PPN) | Checkout · settlement |
| Konfirmasi tindakan berisiko | Pencairan · kembalikan deposit · tolak approval |

---

## 11. Pemetaan halaman → fase

| Fase | Halaman yang lahir |
|---|---|
| 1 | Masuk · pengguna · peran (Admin) |
| 2 | Dompet & mutasi (Agency) · Rekonsiliasi (Admin) |
| 3 | Pembayaran · pencairan · notifikasi |
| 4 | Customer (travel) · Agency · Driver |
| 5 | Pemilik Warung · Penjaga · Grosir |
| 6 | Rental · Charter · transfer penumpang |
| 7 | Promo · referral · level |
| 8 | Halaman operasional tambahan bila perlu |

---

## 12. Lintas peran — dukungan, pelacakan, keamanan akun

Layar yang dipakai lebih dari satu peran dan tidak dimiliki satu vertical saja.

| Halaman | Untuk | Rute web | Status |
|---|---|---|---|
| Tiket dukungan | Pengguna | `/dukungan` · `/dukungan/buat` · `/dukungan/{ticket}` | ✅ D6 |
| Antrean dukungan | Staf platform | `/admin/dukungan` · `/admin/dukungan/{ticket}` | ✅ D6 |
| Pelacakan perjalanan | Penumpang | `/mad-trans/bookings/{booking}/perjalanan` | ✅ D6 |
| Papan pelacakan | Agency | `/mad-trans/agency/{business}/pelacakan` | ✅ D6 |
| MFA / verifikasi dua langkah saat masuk | Semua akun | `/profil/keamanan` · `/masuk/verifikasi` | ✅ D6 |

**Aturan dukungan:** satu tiket = satu percakapan; pengguna hanya melihat tiketnya sendiri;
catatan internal staf tidak pernah dirender di sisi pengguna; tiket tertutup hanya bisa
dibuka kembali oleh staf lewat balasan publik.

**Aturan pelacakan:** tidak ada peta karangan. Aplikasi belum menerima posisi kendaraan,
jadi halaman menampilkan fakta yang tercatat saja — status jadwal, waktu berangkat/selesai,
armada & supir, serta penumpang mana yang sudah naik. Bila kelak ada ping lokasi, halaman
ini yang ditambah, bukan diganti.

**Aturan dua langkah:** kode 6 angka dikirim ke email, disimpan sebagai HMAC (bukan teks),
berlaku 10 menit, sekali pakai, maksimal 5 percobaan. Mengaktifkan wajib mengonfirmasi kode;
mematikan wajib menyertakan kata sandi. Kode **tidak** lewat antrean — kalau worker mati,
tidak ada yang bisa masuk. Pengguna tanpa email tidak bisa mengaktifkan MFA (pesannya jujur,
bukan diam-diam gagal).

