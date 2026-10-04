# 07 — Konsep Mad Warung

> Vertical **utama** GoMad (high frequency). Konsep dimatangkan lewat sesi terpandu.
> Status: **Kelompok 1 selesai · Kelompok 2 diajukan**

---

## 1. Ringkasan konsep

**Mad Warung = intermediary (marketplace) antara Grosir dan Warung retail.**

Masalah yang dipecahkan hari ini: warung harus **antri** di grosir dan **mengambil sendiri** barangnya. Akibatnya warung harus tutup atau ditinggal — padahal warung Madura beroperasi hampir 24 jam.

Yang dihapus Mad Warung: **perjalanan** dan **antrian** — warung tetap buka, tetap dijaga, kulakan selesai dari genggaman.

---

## 2. Masalah & pelaku

**Tiga pesaing sesungguhnya** (bukan sesama aplikasi):

| # | Pesaing | Catatan |
|---|---|---|
| 1 | Warung jalan sendiri ke grosir | Yang paling menderita |
| 2 | **Salesman/canvasser grosir** yang mendatangi warung | **Yang paling berat** — model dominan hari ini |
| 3 | WhatsApp ke grosir langganan | Sudah digital tapi tanpa katalog, harga, atau rekam jejak |

**Implikasi:** karena nomor 2 sudah menghapus antrian & perjalanan untuk banyak warung, Mad Warung **tidak bisa menang hanya dengan "tidak perlu antri"**. Keunggulan harus lebih dalam — dan itu ada di harga.

---

## 3. Posisi bisnis

**Marketplace, bukan distributor.** Grosir tetap penjual; GoMad perantara permintaan.
Konsekuensi: **asset-light** ✅ — tanpa modal kerja barang, tanpa gudang.

---

## 4. Nilai untuk kedua sisi

### Untuk Grosir

| Keuntungan | Catatan penting |
|---|---|
| Dapat pelanggan warung baru | Jangkauan lebih luas |
| **Tidak mengubah cara kerja** | Sudah punya **salesman** untuk antar-jemput, sudah punya **karyawan** untuk menyiapkan pesanan |
| Naik ke ranah digital | Katalog, pesanan, dan rekam jejak jadi digital |

> "Tidak mengubah cara kerja" adalah **kekuatan** tawaran ini — sekaligus **batas** desainnya. Kalau Mad Warung memaksa grosir kerja lembur, mereka akan menolak diam-diam.

### Untuk Warung

| Keuntungan | Penjelasan |
|---|---|
| **Lebih murah** | Di lapangan, warung biasanya ambil dari **agen distributor**, bukan dari **grosir langsung**. Grosir = tier harga lebih rendah |
| **Lebih cepat & praktis** | Warung bisa terus buka & dijaga, tanpa pikir panjang kulakan seperti biasa |
| **Lebih mudah** | Sekali klik, dalam genggaman |
| **Lebih lengkap** | Banyak grosir dalam satu tempat |

---

## 5. Target awal

**Warung sembako — khususnya Warung Madura yang sudah tersebar se-Indonesia.**

Ini pilihan target yang kuat, dan alasannya bukan sekadar "banyak":

| Karakter Warung Madura | Mengapa cocok untuk Mad Warung |
|---|---|
| Beroperasi **hampir 24 jam** | Perjalanan kulakan = **kehilangan penjualan langsung**. Inilah sakit yang paling tajam |
| **Tersebar & mengelompok** | *Local Density Priority* bisa dicapai cepat — justru lewat jaringan sosialnya |
| Jaringan **perantau Madura** yang erat | Kanal distribusi & kepercayaan alami — akuisisi lebih murah dari iklan |
| Volume tinggi, margin tipis | Sangat sensitif harga → "lebih murah" jadi alasan pindah yang nyata |
| Sembako = kebutuhan **harian/mingguan** | Selaras **North Star: Transaction Frequency** |

---

## 6. ~~Pertanyaan yang harus dijawab lebih dulu~~ → **DIKOREKSI, lihat bagian 9**

Klaim **"lebih murah"** adalah tulang punggung tawaran ini. Tapi ada satu rantai logika yang belum tersambung:

> Harga grosir lebih rendah **karena volume**.
> Warung memesan dalam **jumlah kecil**.
> **Bagaimana Mad Warung memberi harga grosir untuk pesanan kecil?**

Tiga kemungkinan jawaban, dan masing-masing mengubah desain secara radikal:

| Kemungkinan | Cara kerja | Konsekuensi |
|---|---|---|
| **A · Agregasi** | GoMad mengumpulkan pesanan banyak warung se-wilayah, lalu membeli dalam volume grosir | Butuh **Local Density** — dan pengiriman jadi **wajib** |
| **B · Perjanjian kanal** | Grosir memberi GoMad harga khusus sebagai mitra kanal | Butuh daya tawar; sulit di awal, mudah setelah volume besar |
| **C · Subsidi platform** | GoMad menutup selisih harga | Tidak berkelanjutan — hanya boleh untuk masa akuisisi |

**Dugaan saya: A**, dan inilah yang menjelaskan dua hal di mindmap-mu yang tadinya terlihat terpisah:

- **`Local Density Priority`** bukan prinsip pemasaran — dia **mekanisme agregasi**. Tanpa warung yang mengelompok, tidak ada volume grosir.
- **`Mad Logistics → Mad Warung Fulfillment` + `Batched & Scheduled Routes`** bukan vertical tambahan — dia **cara mengantar hasil agregasi** (pecah bulk + antar terjadwal per wilayah).

Kalau dugaan ini benar, **Mad Logistics bukan opsional**: dia bagian dari mesin "lebih murah". Ini bertabrakan dengan keputusanmu sebelumnya bahwa Mad Logistics "minimal dulu" di MVP — dan itu wajar, karena waktu itu konsepnya belum sedetail sekarang.

---

## 7. Empat turunan dari pertanyaan di atas

| # | Pertanyaan | Kenapa menentukan |
|---|---|---|
| 1 | **Siapa yang mengantar?** Grosir (salesman) sudah punya rute & jadwal sendiri. Kalau warung pesan di luar jadwal kunjungan, kapan sampai? | Menentukan apakah Mad Logistics wajib ada di MVP |
| 2 | **Dari mana GoMad dapat pendapatan?** Warung marginnya tipis, grosir marginnya juga tipis | Kalau fee diambil dari warung, keunggulan "lebih murah" hilang sendiri |
| 3 | **Warung bayar bagaimana?** Tunai saat kirim, transfer di muka, atau **tempo**? | Tempo = risiko gagal bayar. Siapa yang menanggung? |
| 4 | **Minimum order & frekuensi kirim?** Per hari per wilayah, atau per pesanan? | Menentukan biaya antar per warung — inti unit economics |

---

## 8. Yang sudah pasti

| Item | Keputusan |
|---|---|
| Posisi | Marketplace (intermediary), bukan distributor |
| Pemasok | Grosir existing |
| Peran grosir | Mitra; cara kerja tidak diubah, hanya naik ke digital |
| Target awal | Warung sembako, khususnya Warung Madura |
| Nilai untuk warung | Lebih murah · lebih cepat · lebih mudah · lebih lengkap |
| Prioritas | Vertical **utama** (high frequency) |
| Relasi ke Mad Logistics | **Cukup minimal di MVP** — pengiriman dilakukan grosir atau diambil warung sendiri |
| Sumber pendapatan | fee platform + service fee + tax, **diambil dari warung retail** |
| Minimum order | Diatur sendiri oleh tiap grosir |
| Frekuensi warung Madura | **Minimal 2× sehari** |

---

## 9. Mekanisme harga — HASIL KOREKSI

**Bukan agregasi.** Sumber keunggulan harga sederhana: **grosir memang lebih murah daripada agen distributor.**

```
Hari ini   : principal → distributor → AGEN → warung
             warung membayar harga agen (sudah termasuk margin agen)

Mad Warung : warung membayar HARGA GROSIR + fee GoMad
```

### Pertidaksamaan yang menentukan hidup-mati Mad Warung

```
harga_grosir + fee_GoMad  <  harga_agen
```

| Skenario | Harga agen | Lewat Mad Warung | Hasil |
|---|---|---|---|
| Margin agen **10%** | Rp 110.000 | Rp 108.000 *(grosir 100rb + 3% + flat 5rb)* | Warung hemat Rp 2.000 ✅ |
| Margin agen **5%** | Rp 105.000 | Rp 108.000 | **Lebih mahal** ❌ |

**Kesimpulan strategis:** GoMad **mengambil sebagian margin agen distributor, lalu mengembalikan sisanya ke warung.** Ini bukan subsidi — ini memindahkan margin perantara. Model yang sehat.

**Tapi hidup-matinya bergantung pada satu angka lapangan: selisih harga grosir vs agen.** Angka itu harus kita ketahui.

---

## 10. Pengiriman — DIJAWAB

Tiga jalur, sesuai mindmap:

| Jalur | Padanan di mindmap | Status |
|---|---|---|
| Grosir mengantar sendiri | *Supplier Self-Delivery* | Tersedia sekarang |
| **Warung ambil sendiri** setelah pesanan disiapkan | — | **Mekanisme inti penghapus antrian** |
| Partner / pihak ketiga | *Partner Delivery Network* | Nanti |

**Dugaan agregasi saya salah, dan kamu benar:** Mad Logistics memang **cukup minimal di MVP**. Tidak perlu armada sendiri.

**Catatan penting:** "warung tinggal ambil saat pesanan telah disiapkan" berarti perjalanan **tetap ada**, tapi **antri dan keliling hilang**. Untuk warung 24 jam, itulah yang paling berharga — perjalanan jadi singkat dan pasti, bukan setengah hari.

---

## 11. ⚠️ Dua ketegangan baru

### Ketegangan 1 · Fee flat menghukum frekuensi

Kalau fee diambil dari warung **dan** ada komponen flat per pesanan:

```
Warung Madura pesan 2× sehari = ±60 pesanan/bulan
Komponen flat Rp 5.000 → Rp 300.000/bulan HANYA untuk fee flat
```

Ironinya: **makin sering warung kulakan, makin mahal** — padahal frekuensi tinggi justru **North Star** kita.

> **Rekomendasi:** untuk Mad Warung, pakai **fee berbasis persentase saja** (tanpa flat per pesanan), atau flat **harian/bulanan** — bukan per pesanan. Struktur flat per pesanan cocok untuk Mad Trans (satu tiket sekali jalan), **tidak cocok** untuk kulakan harian.

### Ketegangan 2 · "Grosir tidak mengubah cara kerja" vs katalog digital

Marketplace tidak bisa jalan tanpa **katalog + harga + penerimaan pesanan** yang digital. Sembako = **ratusan SKU** per grosir.

- Kalau **grosir** yang memelihara → itu **mengubah cara kerja** mereka, bertentangan dengan tawaran "tidak berubah"
- Kalau **GoMad** yang memelihara → beban operasional besar (input katalog & update harga ratusan SKU × banyak grosir)

**Pertanyaan yang belum terjawab:** siapa memelihara katalog dan harga? Dan bagaimana grosir menerima pesanan — aplikasi, WhatsApp, atau GoMad yang menyampaikan?

---

## 12. UJI HIDUP-MATI: celah harga hanya 5%

**Angka lapangan (dari kamu):** agen distributor **±5% lebih mahal** daripada grosir.

Artinya seluruh ruang yang bisa diperebutkan GoMad adalah **5%** — dan itu harus menanggung platform fee **dan** PPN sekaligus.

### Simulasi dengan struktur fee sekarang (service fee flat Rp 5.000 + platform 3% + PPN 11%)

| Nilai pesanan | Harga agen | Lewat Mad Warung | Selisih |
|---|---|---|---|
| Rp 200.000 | Rp 210.000 | Rp 212.210 | **+2.210 ❌ lebih mahal** |
| Rp 350.000 | Rp 367.500 | Rp 367.205 | −295 *(nyaris nol)* |
| Rp 500.000 | Rp 525.000 | Rp 522.200 | −2.800 *(0,56%)* |
| Rp 1.000.000 | Rp 1.050.000 | Rp 1.038.850 | −11.150 *(1,06%)* |

**Titik impas: pesanan ±Rp 332.000.** Di bawah itu, warung justru **rugi** memakai Mad Warung.

### Simulasi tanpa komponen flat (platform 3% + PPN saja = 3,33%)

| Nilai pesanan | Harga agen | Lewat Mad Warung | Selisih |
|---|---|---|---|
| Rp 200.000 | Rp 210.000 | Rp 206.660 | −3.340 *(1,6%)* |
| Rp 500.000 | Rp 525.000 | Rp 516.650 | −8.350 *(1,6%)* |
| Rp 1.000.000 | Rp 1.050.000 | Rp 1.033.300 | −16.700 *(1,6%)* |

**Kesimpulan:** tanpa komponen flat, penghematan warung **konsisten 1,6%** di semua ukuran pesanan. Dengan flat Rp 5.000, penghematan itu **hilang untuk pesanan di bawah Rp 332.000** — dan justru hukuman bagi warung yang memesan kecil tapi sering.

### Reframe yang penting

Dengan celah hanya 5%, **harga bukan keunggulan — harga adalah syarat minimum.**

- ❌ Salah: "Mad Warung lebih murah" sebagai nilai jual utama
- ✅ Benar: "Mad Warung **tidak lebih mahal**, dan kamu tidak perlu meninggalkan warung" — keunggulan sesungguhnya adalah **waktu**

Untuk warung yang buka 24 jam dan kulakan 2× sehari, hemat waktu itulah nilai sebenarnya. Tapi janji "tidak lebih mahal" **wajib dijamin** — kalau sekali saja warung merasa lebih mahal, mereka tidak akan kembali.

### Tiga opsi struktur fee untuk Mad Warung

| Opsi | Struktur | Penghematan warung | Catatan |
|---|---|---|---|
| **1** | Platform fee 3% saja, **tanpa** service fee | konsisten **1,6%** | Paling sederhana & aman |
| **2** | Platform fee 3% + service fee **bulanan** (bukan per pesanan) | 1,6% + biaya tetap bulanan | Pendapatan lebih pasti, tidak menghukum frekuensi |
| **3** | Platform fee **2%** + service fee bulanan | **2,5%** | Nilai paling meyakinkan, tapi pendapatan per pesanan lebih kecil |

> **Rekomendasi saya: Opsi 2.** Menjaga tiga komponen pendapatan yang kamu inginkan (platform fee, service fee, tax) **tanpa** menghukum frekuensi tinggi. Service fee diubah basisnya dari *per pesanan* menjadi *per bulan* — itu perbaikan tunggal yang menyelamatkan model.

---

## 13. Pembayaran: transfer di muka × 2 kali sehari = friksi besar

Keputusan: **transfer di muka**. Dengan warung kulakan **2× sehari**:

```
2 transfer/hari × 30 hari = ±60 transfer per warung per bulan
```

**Dua masalah:**

| Masalah | Dampak |
|---|---|
| Friksi bagi warung | 60 kali transfer per bulan — merepotkan, dan bertabrakan dengan janji "sekali klik" |
| Beban rekonsiliasi GoMad | Ops harus mencocokkan 60 transfer ke 60 pesanan, per warung, per bulan |

### Rekomendasi: model saldo (top-up)

```
Warung top-up saldo (mis. sekali seminggu)
     ↓
Setiap pesanan memotong saldo otomatis
     ↓
Warung diberi tahu saat saldo menipis
```

**Keuntungan:**
- Warung: transfer 4× sebulan, bukan 60×
- GoMad: rekonsiliasi jauh lebih ringan — satu top-up, banyak pesanan
- Selaras dengan *North Star*: friksi pembayaran hilang, frekuensi tidak terhambat
- Ledger GoMad **sudah siap** — cukup tambah akun `WARUNG.AVAILABLE`

**Penting:** kalau model saldo dipakai, ledger MVP butuh akun untuk warung. Artinya `owner_type` bertambah (`warung`) — dan karena ledger kita berbasis akun, itu **tambah baris, bukan migrasi**.

---

## 14. Kanal: grosir memakai aplikasi/web GoMad

Keputusan: pesanan diterima grosir **lewat aplikasi/web GoMad**.

**Konsekuensi langsung:**

| Konsekuensi | Penjelasan |
|---|---|
| **Mad Grosir Portal wajib ada di MVP** | Bukan opsional — tanpa ini grosir tidak bisa menerima pesanan |
| "Tidak mengubah cara kerja" perlu ditafsirkan ulang | Yang tidak berubah: cara menyiapkan & mengantar. Yang berubah: pesanan masuk **digital**, bukan dari salesman |
| Peran salesman bergeser | Dari **pencatat pesanan** menjadi **pengantar** — atau tetap keduanya |

### Masih terbuka: siapa memelihara katalog & harga?

Sembako = **ratusan SKU per grosir**. Ini beban terbesar yang belum diputuskan:

| Skenario | Beban GoMad | Kepatuhan grosir |
|---|---|---|
| Grosir memelihara sendiri | Ringan | Berat — ini perubahan cara kerja yang nyata |
| GoMad memelihara | Sangat berat: ratusan SKU × banyak grosir, harga terus berubah | Ringan |
| **Hibrida** — GoMad input awal, grosir update harga | Sedang | Sedang |

---

## 15. Masih terbuka

| # | Pertanyaan | Kenapa menentukan |
|---|---|---|
| 1 | **Struktur fee final Mad Warung** (opsi 1/2/3 di bagian 12) | Menentukan apakah model bertahan di celah 5% |
| 2 | **Model saldo dipakai?** | Menentukan apakah warung jadi pemegang akun ledger di MVP |
| 3 | **Siapa memelihara katalog & harga?** | Beban operasional terbesar yang belum diputuskan |
| 4 | Apakah semua warung target memang beli dari agen? | Yang sudah beli langsung dari grosir tidak dapat manfaat harga sama sekali — hanya hemat waktu |
| 5 | Wilayah percontohan pertama | *Local Density Priority* butuh satu wilayah, bukan "se-Indonesia" |

---

## 16. Sintesis konsep

**Satu kalimat:** Mad Warung adalah marketplace yang membuat warung sembako mendapat **harga grosir** tanpa harus antri atau keliling — dan kulakan 2× sehari jadi semudah sekali klik.

| Aspek | Isi |
|---|---|
| **Model** | Marketplace. Grosir tetap penjual, GoMad perantara permintaan. Asset-light |
| **Alur** | Warung pesan di app → grosir terima di portal → grosir siapkan → diantar grosir / diambil warung → bayar |
| **Target** | Warung sembako, khususnya **Warung Madura** |
| **Frekuensi** | Min. 2× sehari → selaras North Star |

### Nilai untuk warung

| Nilai | Kekuatan |
|---|---|
| **Waktu** | ⭐ **Keunggulan sesungguhnya** — warung tetap buka, tidak ditinggal |
| Kemudahan | Sekali klik |
| Kelengkapan | Banyak grosir dalam satu tempat |
| Harga | ⚠️ **Syarat minimum, bukan keunggulan** — celah hanya 5% |

### Nilai untuk grosir

Dapat pelanggan warung baru · naik ke digital · **cara kerja inti tidak berubah** (tetap punya salesman untuk antar, karyawan untuk menyiapkan).

### Konsekuensi arsitektur

| Konsekuensi | Status |
|---|---|
| **Mad Grosir Portal wajib ada di MVP** | Karena grosir menerima pesanan lewat platform |
| **Warung jadi pemegang akun ledger** | Jika model saldo dipakai (`WARUNG.AVAILABLE`) |
| Mad Logistics cukup minimal | Tidak perlu armada sendiri |

### Dua keputusan konsep yang masih terbuka

1. **Basis service fee** — per pesanan (menghukum frekuensi) atau bulanan (aman). *Rekomendasi: bulanan.*
2. **Model saldo top-up** — dipakai atau tidak. *Rekomendasi: dipakai.*

---

## 17. LAPISAN BARU: struktur pengguna Mad Warung

Dari paparanmu, Mad Warung bukan hubungan satu-ke-satu antara "warung" dan "grosir". Ada **tiga lapis**:

```
PEMILIK WARUNG          → bisa memiliki 1 atau lebih warung
      │
      ▼
WARUNG (outlet)         → unit fisik; punya saldo & aktivitas sendiri
      │
      ▼
PENJAGA WARUNG          → akun sendiri · punya jadwal shift · melakukan kulakan
```

**Semua aktivitas penjaga tercatat dan terlihat oleh pemilik.**

### Mengapa ini penting — proposisi nilainya naik kelas

| Rumusan sebelumnya | Dengan lapisan ini |
|---|---|
| Lebih murah | ✓ tetap |
| Lebih cepat & praktis | ✓ tetap |
| Lebih mudah | ✓ tetap |
| Lebih lengkap | ✓ tetap |
| — | ⭐ **Kendali atas usaha 24 jam yang tidak dia jaga sendiri** |

Alasannya sederhana: warung Madura buka ~24 jam → **pemilik tidak mungkin menjaga sendiri** → penjaga yang menjalankan, termasuk kulakan.

Artinya pemilik menyerahkan **uang dan stok** ke tangan orang lain, di lokasi yang tidak dia awasi. Yang dia butuhkan bukan cuma harga grosir — tapi **visibilitas dan kendali**.

> **Pergeseran posisi:** Mad Warung pindah dari *alat belanja* menjadi **alat pengawasan usaha**.
> Ini proposisi yang jauh lebih kuat ke investor, dan lebih tahan terhadap tekanan harga — karena pemilik bersedia membayar untuk **ketenangan**, bukan hanya untuk diskon 1,6%.

### Konsekuensi ke desain

| Konsekuensi | Catatan |
|---|---|
| **Identitas berlapis** | Pemilik · Warung · Penjaga = tiga entitas berbeda. Sejalan dengan prinsip *Unified User Identity* |
| **Multi-outlet** | Satu pemilik → banyak warung. Laporan harus bisa **per warung** dan **gabungan** |
| **Penjaga = akun nyata** | Bukan sekadar catatan. Dia login, melihat katalog, dan bertransaksi |
| **Jadwal shift** | Menentukan siapa yang bertanggung jawab pada jam berapa |
| **Jejak aktivitas** | Setiap kulakan terhubung ke **penjaga yang melakukannya** — bisa diaudit |
| **Saldo** | Perlu diputuskan: di tingkat pemilik (satu saldo banyak warung) atau **per warung** |
| **"Grosir terdekat"** | Pemilihan grosir berbasis lokasi |

### Pertanyaan konsep yang muncul

| # | Pertanyaan | Kenapa menentukan |
|---|---|---|
| 1 | **Kewenangan penjaga** — bebas kulakan, atau ada batas/approval pemilik? | Ini menentukan apakah butuh *spending limit* — pola yang sama dengan deposit gating di Mad Trans |
| 2 | **Jadwal shift** — hanya untuk akuntabilitas, atau juga menentukan batas kewenangan per shift? | Menentukan apakah jadwal ikut menggerakkan izin transaksi |
| 3 | **Saldo** — dipegang pemilik (satu saldo, banyak warung) atau per warung? | Menentukan bentuk akun ledger & pelaporan |
| 4 | Satu penjaga bisa menjaga beberapa warung? | Menentukan relasi penjaga ↔ warung: 1:N atau M:N |
| 5 | **"Grosir terdekat"** — dipilih otomatis dari lokasi, atau warung memilih sendiri dari daftar? | Menentukan apakah butuh mesin pencocokan geografis |
