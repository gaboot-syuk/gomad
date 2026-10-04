# Blueprint 09 — Lingkungan Docker & Verifikasi Live

> **Aturan baru:** setiap fase **ditutup dengan verifikasi live** — aplikasi dijalankan lalu ditelusuri
> di browser preview editor. **Bukan** hanya mengandalkan folder `/tests/`.
> Dokumen ini menjelaskan lingkungannya, cara kerjanya, dan batas jujurnya.

---

## 1. Hasil pengecekan lingkungan (2026-10-04)

| Komponen | Status |
|---|---|
| Docker CLI | ✅ 28.5.2 |
| Docker Compose | ✅ 2.40.3 |
| Docker daemon | ✅ Berjalan · tanpa sudo (grup `docker`) |
| Storage driver | overlay2 |
| Host | Kali GNU/Linux (container) |
| Lingkungan | **WSL2** — kernel 6.18.40.1-microsoft-standard-WSL2 |
| Flutter | ✅ 3.44.1 · Dart 3.12.1 |
| Browser lokal | ✅ google-chrome · chromium · firefox |
| Browser preview editor | ✅ Buka · baca · tangkap layar · interaksi |

**Port yang sudah terpakai proyek lain:** `8100` `8025` `1025` — **jangan dipakai GoMad.**

---

## 2. Stack Docker GoMad

```
docker/
├── compose.yaml
├── php/            Dockerfile PHP-FPM + ekstensi
├── nginx/          konfigurasi virtual host
└── mysql/          konfigurasi + volume data
```

| Layanan | Fungsi | Port host (usulan) |
|---|---|---|
| `nginx` | Web server — pintu masuk semua request | **8200** |
| `app` | PHP-FPM (Laravel) | internal |
| `queue` | Worker antrean (Redis) | internal |
| `scheduler` | Tugas terjadwal (settlement, pelepasan kursi) | internal |
| `db` | Database MariaDB sementara (hanya profil pemulihan backup) | tidak diekspos |
| `redis` | Cache · antrean · sesi | **6380** |
| `mailpit` | Menangkap email keluar saat pengujian | **8125** |
| `vite` | Dev server aset (hot reload) | **5273** |

> Aplikasi lokal memakai **Aiven MySQL via TLS terverifikasi**; database
> MariaDB hanya dinyalakan untuk uji restore backup. Backup harian disimpan
> lokal dan disalin ke bucket privat Cloudflare R2. Redis dan Mailpit lokal.

**Semua lewat `docker compose up`.** Tidak ada langkah pemasangan manual di luar Docker, supaya lingkungan bisa dihancurkan dan dibangun ulang kapan saja.

---

## 3. Alur kerja harian

```
1. docker compose up -d           → seluruh layanan berjalan
2. bash bin/compose exec app php artisan migrate --seed
3. Buka http://localhost:8200     → di browser preview editor
4. Telusuri sesuai peran
5. docker compose down            → berhenti bersih
```

**Perintah yang wajib ada sejak Fase 0:**

| Perintah | Fungsi |
|---|---|
| `compose up` | Nyalakan seluruh stack |
| `bash bin/compose exec app php artisan migrate --seed` | Migrasi aditif ke Aiven via TLS + data contoh |
| `bash bin/test` | Jalankan tes terisolasi pada SQLite dalam memori |
| `bash bin/compose exec app php artisan app:reset-demo` | **Kembalikan keadaan demo**; ditolak jika target MySQL remote |
| `bash bin/verify-backup-restore [--r2-latest]` | Pulihkan dump lokal atau R2 ke MariaDB disposable |
| `bash bin/compose logs -f` | Lihat log layanan |

`migrate:fresh --seed` hanya untuk CI atau database disposable lokal; jangan
jalankan pada Aiven. Kredensial tidak disalin ke `.env`. Pemulihan backup selalu
memakai database MariaDB terpisah dan membuang schema sementara setelah
verifikasi.

---

## 4. Protokol verifikasi live — Web

Dilakukan **setiap fase, sebelum fase dinyatakan selesai.**

| # | Langkah |
|---|---|
| 1 | `docker compose up -d` · pastikan semua layanan sehat |
| 2 | Jalankan migrasi + data contoh |
| 3 | Buka aplikasi di **browser preview editor** |
| 4 | **Login sebagai setiap peran** yang terlibat di fase itu |
| 5 | Telusuri alur utama **sampai selesai** — bukan sekadar halaman terbuka |
| 6 | Tangkap layar setiap langkah penting (bukti visual) |
| 7 | Uji juga **keadaan gagal**: input salah, akses tanpa izin, data kosong |
| 8 | Uji **ukuran layar HP** untuk halaman yang akan dipakai penjaga (responsif) |
| 9 | Catat temuan → perbaiki → **ulangi dari langkah 1** |

**Yang membuat verifikasi ini bernilai:** langkah 5, 7, dan 8. Membuka halaman itu bukan verifikasi — menyelesaikan alur, mencoba yang salah, dan melihat di layar kecil itu verifikasi.

### Yang bisa diuji live — dan sering terlewat

| Skenario | Cara menguji live |
|---|---|
| **Kursi dobel** | Buka dua tab, pesan kursi yang sama **hampir bersamaan** |
| **Callback duplikat** | Kirim ulang webhook yang sama **3×** dari terminal, lalu periksa saldo |
| **Otorisasi bocor** | Login sebagai usaha A, coba buka data usaha B lewat URL langsung |
| **Sesi kedaluwarsa** | Tunggu sesi habis, pastikan pengguna dialihkan dengan benar |

---

## 5. Protokol verifikasi live — Mobile native

Karena preview editor menampilkan **browser**, bukan layar Android, verifikasi mobile memakai **dua langkah**:

| # | Langkah | Membuktikan |
|---|---|---|
| 1 | `flutter build apk --debug` | Kode **benar-benar terkompilasi** untuk Android |
| 2 | `flutter analyze` | Tidak ada kesalahan statis |
| 3 | `flutter run -d web-server --web-port 8300` | **Verifikasi visual & alur** di browser preview |
| 4 | Buka `http://localhost:8300` di browser preview editor | Tampilan & alur bisa ditelusuri seperti aplikasi |
| 5 | Tangkap layar · uji alur utama | Bukti visual per fase |

**Urutannya penting:** **build dulu, baru test live** — sesuai permintaanmu. Kalau langkah 1 gagal, tidak ada gunanya lanjut.

### Batas jujur yang perlu diketahui

Langkah 3–4 menjalankan **target web** dari kode Flutter — bukan target Android. Jadi ia membuktikan **tampilan dan alur**, tapi **tidak** membuktikan perilaku khas perangkat:

| Tidak terbukti lewat web preview | Cara menutupnya |
|---|---|
| Kamera, GPS, penyimpanan lokal | Uji di emulator/perangkat nyata |
| Notifikasi push | Uji perangkat nyata |
| Interaksi sentuh & gestur | Uji perangkat nyata |
| Izin aplikasi (permission) | Uji perangkat nyata |

**Kesimpulan:** web preview menangkap mayoritas kesalahan (tampilan, alur, logika) dengan cepat — tapi **perilaku khas perangkat tetap butuh emulator atau HP sungguhan.** Ini akan dimasukkan sebagai langkah wajib di Fase 9.

---

## 6. ⚠️ Batas verifikasi live — dan apa yang menutupinya

Verifikasi live **ditambahkan**, bukan menggantikan tes otomatis. Alasannya bukan formalitas:

| Pertanyaan | Bisakah dijawab live test? | Yang bisa menjawab |
|---|---|---|
| Aplikasi jalan & alurnya benar? | ✅ Ya — justru inilah kekuatannya | Verifikasi live |
| Idempotency callback | ⚠️ Sebagian — bisa dikirim manual 3× | Tes otomatis + verifikasi live |
| **Rekonsiliasi ledger tanpa selisih** | ❌ Tidak — butuh **semua** skenario | **Tes otomatis** |
| Race condition skala besar | ❌ Tidak — hanya 2 tab | Tes otomatis |
| Perilaku saat gagal di tengah transaksi | ⚠️ Sulit dipicu manual | Tes otomatis |

**Aturan yang dipakai:**

> **Fase selesai = tes otomatis hijau DAN verifikasi live berhasil.**
> Bukan salah satu. Keduanya menjawab pertanyaan yang berbeda.

Analogi sederhananya: verifikasi live membuktikan **rumahnya bisa ditinggali**. Tes uang membuktikan **fondasinya tidak retak**. Kamu tidak bisa memilih salah satu.

---

## 7. Yang berubah di rencana fase

| Sebelumnya | Sekarang |
|---|---|
| Fase ditutup dengan tes otomatis | Fase ditutup dengan **tes otomatis + verifikasi live** |
| Lingkungan disiapkan bertahap | **Docker disiapkan di Fase 0** — sebelum fitur apa pun |
| Verifikasi mobile di Fase 9 | **Build + web preview dijalankan sejak alur pertama ada** |

**Perubahan di Fase 0:** menambah — Docker stack berjalan · perintah `compose` lengkap · `app:reset-demo` tersedia · **verifikasi live pertama berhasil** (buka halaman kosong di browser preview).

**Gerbang terakhir Fase 0 (lulus · 2026-10-04):** setelah seluruh alur lokal,
backup, CI, dan sinkronisasi kontrak API→mobile lulus, staging Render free tier
berhasil dijalankan di <https://gomad-api-staging.onrender.com>. Health check
`/api/v1/health` mengembalikan HTTP 200; browser preview membuka halaman melalui
HTTPS tanpa mixed content. Konfigurasinya memakai SQLite in-memory dan tidak
terhubung ke Aiven. Jangan letakkan kredensial deployment di repo atau chat.
