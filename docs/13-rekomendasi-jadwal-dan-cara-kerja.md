# 13 — Rekomendasi Jadwal & Cara Kerja

> Menjawab: kendala apa yang harus dipakai, dan bagaimana mengerjakan lingkup **rilis lengkap** tanpa membuatnya melebar.
> Prinsip yang dipakai: yang paling menentukan bukan jumlah fitur, tapi **jumlah wadah yang harus dirawat**.

---

## 1. Kendala: rekomendasi

**Jangan pakai "kapan selesai" sebagai kendala. Pakai `runway` (berapa lama dana cukup).**

Alasannya: tanggal selesai tidak bisa diprediksi sebelum kapasitas diketahui, sedangkan runway adalah angka yang **sudah** bisa dihitung. Runway juga mengikat secara nyata.

| Kendala | Layak dipakai? | Catatan |
|---|---|---|
| **Runway / dana** | ⭐ **Ya — jadikan kendala utama** | Angka nyata, mengikat, memaksa prioritas |
| **Tonggak eksternal** (demo investor, onboarding mitra pertama) | ⭐ Ya | Komitmen ke orang lain jauh lebih mengikat daripada komitmen ke diri sendiri |
| Batas kalender tanpa dasar | ❌ | Menghasilkan jadwal fiktif |
| Tanpa kendala | ❌ | Proyek melebar tanpa akhir |

**Rekomendasi konkret — tetapkan dua tonggak eksternal:**

```
Tonggak A · tanggal demo      → cukup untuk dipamerkan
Tonggak B · tanggal uji nyata → cukup untuk dipakai warung & grosir sungguhan
```

---

## 2. Tiga tingkat "selesai" yang berbeda

Ini praktik terbaik yang paling sering terlewat. **"Selesai" punya tiga arti berbeda**, dan mencampurnya membuat jadwal kacau.

| Tingkat | Artinya | Boleh ada apa | Gelombang |
|---|---|---|---|
| **Demo-able** | Bisa dipamerkan tanpa malu | Data contoh · satu jalur bahagia · belum tahan salah | 1–2 |
| **Pilot-able** | Bisa dipakai warung & grosir nyata di **satu wilayah** | Uang benar · rekonsiliasi bisa ditelusuri · error ditangani | 3 |
| **Release-able** | Lengkap, stabil, aman uang | Semua fitur · siap dibuka publik | 6 |

**Aturan:** jangan pernah membuka tingkat berikutnya sebelum yang sebelumnya benar-benar selesai. Membuka onboarding massal sebelum alur uang terbukti adalah cara paling mahal untuk menemukan bug.

---

## 3. ⭐ Cara mengecilkan pekerjaan tanpa memotong fitur

Saya sebelumnya menghitung "7 permukaan aplikasi". Itu **7 permukaan**, tapi bukan berarti **7 aplikasi**.

> **Praktik terbaik: pisahkan menurut PERAN, bukan menurut PRODUK.**

Lihat pola yang sudah kita temukan di `docs/09`:

| Peran | Pemakai | Wadah |
|---|---|---|
| **Pihak Luar** | Penumpang · Penyewa · Pembeli | 📱 **Aplikasi GoMad** (1) |
| **Staff** | Penjaga warung · Driver | 📱 **Aplikasi GoMad Mitra** (2) |
| **Pemilik Usaha** | Pemilik warung · Pemilik travel/rental · **Grosir** | 💻 **Web GoMad** (3) |
| **Admin/Ops** | Internal GoMad | 💻 Web GoMad (peran terpisah) |

### Hasilnya: dari 7 permukaan → **3 wadah**

| Wadah | Teknologi | Melayani peran |
|---|---|---|
| **Web GoMad** | Laravel + Inertia/Blade | Portal Pemilik Warung · Portal Grosir · Admin Console |
| **Aplikasi GoMad** | Flutter | Customer (travel & rental) |
| **Aplikasi GoMad Mitra** | Flutter | Penjaga warung · Driver (mode berbeda) |

**Yang tidak berubah:** semua fitur tetap dibangun. Yang berkurang hanya **jumlah aplikasi yang harus dirawat, dirilis, dan diperbarui** — dan itu biasanya sumber biaya terbesar, bukan penulisan kodenya.

**Kenapa pola peran bisa dipakai:** karena kita sudah membuktikan di `docs/09` bahwa *Penjaga Warung* dan *Driver* secara struktur identik — keduanya staff dari seorang pemilik usaha. Jadi satu wadah dengan mode berbeda itu wajar, bukan paksaan.

> ⚠️ **Catatan:** keputusan ini menggantikan saran awal saya di sesi awal (aplikasi terpisah per peran). Waktu itu lingkupnya belum diketahui; sekarang, dengan rilis lengkap dan sumber daya terbatas, **menggabungkan wadah jauh lebih rasional.**

---

## 4. Praktik terbaik pengerjaan

### a. Ledger lebih dulu — dan diuji lebih dulu

Ledger bukan fitur, dia **fondasi**. Kalau salah, semua di atasnya salah dan harus dibongkar. Bangun dan ujilah sebelum vertical apa pun menyentuhnya.

### b. Rangka tulang berjalan (*walking skeleton*)

Jangan bangun per lapisan. Bangun **satu alur tertipis yang benar-benar jalan dari ujung ke ujung**, lalu tebalkan.

```
Contoh: cari jadwal → pilih kursi → bayar → e-ticket → settlement ke agency
```

Tipis, tapi **lengkap**. Ini menyingkap integrasi yang rusak di minggu pertama, bukan di bulan kelima.

### c. Rekonsiliasi manual sejak transaksi pertama

Meski baru ada 3 transaksi, catat di lembar terpisah dan cocokkan dengan ledger setiap hari. **Ini menemukan bug uang lebih cepat daripada tes otomatis.** Tes membuktikan yang kamu pikirkan; rekonsiliasi menemukan yang tidak kamu pikirkan.

### d. Pilot dengan satu agency & satu grosir

Jangan buka onboarding sebelum alur uang terbukti pada satu mitra sungguhan yang sabar. Kalau ada masalah, malu ke satu orang lebih murah daripada malu ke seratus.

### e. Feature freeze sebagai gelombang tersendiri

Rilis **bukan** saat kode terakhir selesai. Sisa waktu untuk memperbaiki bug uang, menyelesaikan rekonsiliasi, dan menulis prosedur operasional. Beri porsi khusus — jangan dianggap sisa waktu.

### f. Satu wilayah dulu — *Local Density Priority*

Jangan uji nasional. Konsekuensi operasional dari wilayah yang terlalu luas jauh lebih mahal daripada konsekuensi teknis.

---

## 5. Kerangka jadwal: gelombang → tonggak

| Gelombang | Isi | Tingkat selesai |
|---|---|---|
| **0** | Blocker + keputusan tersisa | — |
| **1 · Core** | Ledger · Identity · Access · Settlement · Payment · Notification | **Demo-able** |
| **2 · Mad Warung inti** | Portal Pemilik · Halaman Penjaga · Portal Grosir | **Pilot-able** · 🎯 **Tonggak A** |
| **3 · Mad Trans inti** | Travel + dompet agency + pencairan + COD & deposit gating | Demo-able |
| **4 · Mad Trans lanjutan** | Rental · Charter · multi-stop per segmen · transfer penumpang · OTS · deposit penyewa | Pilot-able |
| **5 · Keterlibatan** | Promo · Referral · Level & Misi | Release-able |
| **6 · Penutup** | *Feature freeze* · perbaikan · rilis store · prosedur operasional | **Release-able** |

### Referensi kasar waktu

⚠️ **Asumsi: 1 orang, purna waktu, sudah mahir Laravel & Flutter.** Angka ini untuk memberi gambaran skala, bukan janji.

| Gelombang | Perkiraan |
|---|---|
| 1 · Core | 4–8 minggu |
| 2 · Mad Trans inti | 6–10 minggu |
| 3 · Mad Warung | 6–10 minggu |
| 4 · Mad Trans lanjutan | 6–10 minggu |
| 5–6 · Keterlibatan & penutup | 3–6 minggu |
| **Total** | **±6–10 bulan** |

**Catatan penting:** dua gelombang **tidak bisa dipercepat dengan menambah orang** — gelombang 1 (ledger) dan gelombang 6 (stabilisasi). Sisanya bisa diparalelkan sebagian kalau ada tim.

Kalau 6–10 bulan terlalu panjang, tuas yang tersedia cuma tiga: **tambah tenaga**, **kurangi lingkup**, atau **geser Tonggak**. Tidak ada tuas keempat.

---

## 6. Yang saya butuhkan untuk membuat jadwal nyata

| # | Informasi | Untuk apa |
|---|---|---|
| 1 | **Siapa yang mengerjakan** — kamu sendiri, atau ada tim? | Menentukan kapasitas |
| 2 | **Berapa jam per minggu** | Menentukan durasi |
| 3 | **Runway** — berapa bulan dana cukup | Menentukan gelombang mana yang realistis |
| 4 | **Tanggal tonggak eksternal** (kalau ada) | Menentukan titik potong |
| 5 | **Tingkat kemahiran** di Laravel & Flutter | Menentukan asumsi kecepatan |

Tanpa lima ini, jadwal apa pun hanya angka karangan. Dengan lima ini, kerangka gelombang di atas bisa saya ubah jadi jadwal bertanggal.
