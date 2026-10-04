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

---

## 2. Pihak Luar — Customer

📱 **Mobile-first** · Fase 4 (travel) & 6 (rental)

| # | Halaman | Isi penting |
|---|---|---|
| 1 | Cari jadwal | Rute · tanggal · jumlah penumpang |
| 2 | Hasil pencarian | Bandingkan jadwal antar agency · harga per segmen transparan |
| 3 | Pilih kursi | Denah kursi · kursi supir terkunci · kursi terisi tampak jelas |
| 4 | Checkout | Alamat jemput & antar · data penumpang · promo |
| 5 | Pembayaran | Metode utama + **fallback QRIS/VA** bila auto-charge gagal |
| 6 | E-ticket | Kode, kursi, detail, bisa ditunjukkan ke supir |
| 7 | Pesanan saya | Daftar + detail + status |
| 8 | Pembatalan | Estimasi refund sebelum konfirmasi — **bukan setelah** |
| 9 | Rental | Cari kendaraan · kalender ketersediaan · unggah KTP/SIM/selfie |
| 10 | Profil & notifikasi | |

**Alur wajib berhasil:** cari → pilih kursi → bayar → e-ticket terbit.

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

---

## 4. Staff — Driver

📱 **Mobile-first** · Fase 4

| # | Halaman | Isi penting |
|---|---|---|
| 1 | Jadwal hari ini | Daftar tugas |
| 2 | Detail jadwal | Daftar penumpang & titik jemput |
| 3 | Mulai / selesai perjalanan | |
| 4 | Konfirmasi COD | Tandai penumpang sudah membayar tunai |
| 5 | Ajukan transfer penumpang | Bila ada penumpang yang perlu dialihkan |

---

## 5. Pemilik Usaha — Pemilik Warung

💻 **Desktop-first** · Fase 5

| # | Halaman | Isi penting |
|---|---|---|
| 1 | Dasbor | Ringkasan **semua outlet** |
| 2 | Outlet | Tambah & kelola warung |
| 3 | Penjaga | Undang · aktif/nonaktif · detail |
| 4 | **Plafon & batas** | Batas harian · batas per pesanan · ambang approval |
| 5 | **Jadwal shift** | Siapa bertugas kapan |
| 6 | **Approval** | Permintaan menunggu · setujui/tolak **+ alasan** |
| 7 | Pesanan | Per outlet · filter status |
| 8 | Laporan | Per outlet **dan** gabungan · aktivitas per penjaga |

**Alur wajib berhasil:** approval dari jarak jauh saat penjaga melewati plafon.

---

## 6. Pemilik Usaha — Grosir

💻 **Desktop-first** · Fase 5

| # | Halaman | Isi penting |
|---|---|---|
| 1 | Dasbor | Pesanan hari ini |
| 2 | Katalog & harga | Produk · harga **per grosir** · minimum order · stok |
| 3 | Pesanan masuk | Konfirmasi · tandai siap |
| 4 | Riwayat & settlement | Rincian penyetoran |

---

## 7. Pemilik Usaha — Agency (travel & rental)

💻 **Desktop-first** · Fase 4 & 6

| # | Halaman | Isi penting |
|---|---|---|
| 1 | Dasbor | Ringkasan penjualan & jadwal |
| 2 | Armada | Kendaraan · **denah kursi** |
| 3 | Rute | Titik pemberhentian · **harga per segmen** |
| 4 | Jadwal | Buat · terbitkan · **setujui pengajuan supir** |
| 5 | Pengemudi | Tugaskan ke jadwal |
| 6 | Pesanan & penumpang | Daftar · detail · transfer penumpang |
| 7 | Rental | Kendaraan · dokumen penyewa · serah terima |
| 8 | **Dompet** | Saldo tersedia & jaminan · mutasi · hold aktif |
| 9 | Settlement & pencairan | Ajukan · riwayat |
| 10 | Laporan | |

**Alur wajib berhasil:** terbitkan jadwal → terima pesanan → dana masuk dompet → cairkan.

---

## 8. Admin / Ops

💻 **Desktop** · Fase 1–3

| # | Halaman | Isi penting |
|---|---|---|
| 1 | Dasbor operasional | Kesehatan sistem & antrean |
| 2 | Verifikasi | Mitra & dokumen (KTP/SIM/selfie) |
| 3 | Pengguna & peran | |
| 4 | **Keuangan** | Settlement · pencairan · refund · **rekonsiliasi** |
| 5 | Parameter platform | Fee · timeout · batas |
| 6 | Sengketa | Penanganan keluhan |
| 7 | Log audit | Siapa melakukan apa |

> **Halaman rekonsiliasi adalah yang terpenting di sini.** Kalau ops tidak bisa mencocokkan uang dengan cepat, setiap selisih jadi krisis.

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
