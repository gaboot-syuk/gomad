# Blueprint 14 — Peta komponen web ke widget Flutter

> **Fase 9.** Aplikasi mobile tidak menyalin logika bisnis; yang dipindahkan hanya
> tampilan. Karena itu setiap komponen web dipetakan ke satu widget (atau pola)
> yang sudah ada di Material 3, bukan direka ulang per layar.
>
> Rujukan: `blueprint/03` (kontrak API) · `blueprint/06` (Fase 9) · `docs/05` (Core)

Implementasi acuan: `mobile/lib/`.

---

## 1. Prinsip

| # | Prinsip | Konsekuensi |
|---|---|---|
| 1 | **Server tetap satu-satunya tempat aturan** | Widget hanya mengirim input dan menampilkan hasil; tidak ada perhitungan harga/diskon di aplikasi |
| 2 | **Satu komponen web = satu pola widget** | Halaman baru tidak menambah gaya baru |
| 3 | **Klien dari generator** | Model & pemanggilan API berasal dari `gomad_api`; tidak ada `http.post` manual untuk domain |
| 4 | **Uang & tanggal diformat di satu tempat** | `lib/src/format.dart` — Rupiah tanpa desimal, tanggal ISO untuk API |
| 5 | **Aksesibilitas sama dengan web** | Label teks, bukan hanya warna/ikon; tombol minimal 48 dp |

---

## 2. Peta komponen

| Komponen web (Blade) | Widget Flutter | Catatan |
|---|---|---|
| Kartu ringkasan (`gm-card`) | `Card` + `Padding(16)` | Sudut 12, tanpa bayangan berlebih |
| Tabel rincian harga | `Column` berisi `Row` label–nilai | Label rata kiri, nominal rata kanan, `monospace` opsional |
| Tombol utama (`btn-primary`) | `FilledButton` | Satu per layar |
| Tombol sekunder | `OutlinedButton` | Aksi seperti "Salin tautan" |
| Form login/registrasi | `TextField` + `FilledButton` | `obscureText` untuk sandi, `key` tetap untuk tes |
| Pilihan tanggal | `showDatePicker` | Rentang 90 hari ke depan |
| Daftar hasil pencarian | `ListView.separated` | `ListTile` + nominal sebagai `trailing` |
| Peta kursi | `Wrap` berisi `ChoiceChip` | Kursi terisi: `onSelected: null` + ikon blok |
| Badge status | `Chip` / teks berwarna semantik | Tidak memakai warna saja — selalu ada teks |
| Notifikasi / peringatan | `SnackBar` | Untuk konfirmasi singkat |
| Galat validasi form | Teks di bawah field | Pesan dari server ditampilkan apa adanya |
| Paginasi | Tarik-untuk-muat (`RefreshIndicator`) | Batas 20 per halaman mengikuti kontrak |

---

## 3. Struktur folder

```
mobile/
├── api_client/          # hasil generate — JANGAN diedit
├── contracts/           # salinan kontrak OpenAPI (di-commit)
├── lib/
│   ├── main.dart        # kerangka aplikasi; tema mengikuti peran
│   └── src/
│       ├── api_config.dart    # base URL + Dio + klien
│       ├── session.dart       # token, refresh otomatis, retry 401
│       ├── token_store.dart   # penyimpanan token (memori → aman di produksi)
│       ├── workspace.dart     # account_type → ruang kerja native
│       ├── format.dart        # Rupiah, jam, tanggal
│       ├── theme/
│       │   ├── uds.dart         # token UDS v1.0 (salinan dari web)
│       │   └── gomad_theme.dart # dua bahasa desain → ThemeData
│       └── ui/
│           ├── kit/gm_kit.dart  # padanan `x-gm.*`
│           ├── shell/app_shell.dart
│           ├── login_page.dart  home_page.dart
│           ├── seat_page.dart   payment_page.dart  account_page.dart
│           ├── passenger/booking*.dart
│           ├── driver/tasks_page.dart  driver/task_detail_page.dart
│           └── warung/pos_page.dart  sales_report_page.dart
└── tool/pilot_smoke.dart     # uji alur pilot langsung ke server
```

---

## 4. Alur per peran

### 4.1 Penumpang

```
Login → Cari jadwal → Pilih kursi + data penumpang → Pesan → Bayar (Snap)
      → Pesanan saya → Rincian (batalkan bila masih bisa)
```

| Langkah | Endpoint |
|---|---|
| Login | `POST /v1/auth/login` (token 30 hari, nama perangkat dicatat) |
| Profil | `GET /v1/me` |
| Perpanjang sesi | `POST /v1/auth/refresh` (token berotasi) |
| Cari jadwal | `GET /v1/mad-trans/schedules` |
| Peta kursi | `GET /v1/mad-trans/schedules/{id}/seats` |
| Pesan | `POST /v1/mad-trans/bookings` (`Idempotency-Key` per percobaan) |
| Bayar | `POST /v1/mad-trans/bookings/{id}/pay` (Snap token + redirect URL) |
| Pesanan saya | `GET /v1/mad-trans/bookings` · `GET /v1/mad-trans/bookings/{id}` |
| Batalkan | `POST /v1/mad-trans/bookings/{id}/cancel` |

### 4.2 Supir

```
Tugas saya → Rincian tugas → [Mulai perjalanan] → boarding per penumpang
           → [Selesaikan perjalanan]
```

| Langkah | Endpoint |
|---|---|
| Daftar tugas | `GET /v1/mad-trans/driver/schedules` (aktif + selesai 7 hari) |
| Rincian | `GET /v1/mad-trans/driver/schedules/{id}` |
| Mulai / selesai | `POST /v1/mad-trans/schedules/{id}/start` · `/finish` |
| Boarding / tidak naik | `POST /v1/mad-trans/bookings/{b}/passengers/{p}/board` · `/no-show` |

Satu aksi utama per keadaan: `published` → "Mulai perjalanan", `in_progress` →
boarding lalu "Selesaikan perjalanan", `completed` → tidak ada aksi.
`can_board` dari server hanya menyalakan/mematikan tombol; penolakan server
ditampilkan apa adanya.

### 4.3 Penjaga warung (kasir)

```
Kasir → ketuk barang → ± jumlah → Bayar → uang diterima → kembalian → Simpan
```

| Langkah | Endpoint |
|---|---|
| Barang + angka hari ini | `GET /v1/warung/businesses/{id}/pos` |
| Simpan penjualan | `POST /v1/warung/businesses/{id}/pos/sales` |
| Riwayat (pemilik) | `GET /v1/warung/businesses/{id}/pos/sales` |

Warung yang dipakai diambil dari `/v1/me/memberships` (tipe `warung`), bukan
input bebas. Total dan kembalian yang tampil hanya untuk mata; angka yang
tersimpan tetap dihitung server dari `retail_listings`.

Uji cepat tanpa perangkat:

```
cd mobile
dart run tool/pilot_smoke.dart
```

---

## 5. Yang belum ada (sengaja ditunda)

| Item | Alasan |
|---|---|
| `flutter_secure_storage` | Butuh plugin native; `TokenStore` sudah disiapkan sebagai titik ganti |
| Push notification (FCM) | Menunggu kredensial Firebase |
| Buka tautan Snap langsung | Butuh `url_launcher`; sementara tautan bisa disalin |
| Bundling font Inter | web pun jatuh ke font sistem; berkas font belum ada di repo |

---

## 6. Status kriteria Fase 9

| Kriteria | Status | Bukti |
|---|---|---|
| Klien Dart otomatis dari kontrak | ✅ | `bin/generate-dart-client` (27 kelompok API, ±390 model, `dart analyze` tanpa error) |
| Auth token & refresh bekerja | ✅ | `test/session_test.dart` + `tool/pilot_smoke.dart` ke server lokal |
| Alur penumpang | ✅ | Beranda, pesanan, rincian, pembatalan |
| Alur supir | ✅ | Daftar tugas, rincian, boarding, mulai/selesai |
| Alur penjaga warung | ✅ | Kasir + riwayat nota untuk pemilik |
| Tema UDS (dua bahasa desain) | ✅ | `theme/uds.dart`, `theme/gomad_theme.dart`, `workspace.dart` |
| Tidak ada logika bisnis diduplikasi | ✅ | Semua perhitungan di Core Laravel; aplikasi hanya memformat tampilan |
| Kerangka `android/` terpasang | ✅ | `flutter create . --platforms=android --org id.gomad`; izin INTERNET dipindah ke manifest utama |
| APK terbangun | ✅ | `app-debug.apk` (141 MB) dan `app-release.apk` (52 MB) di `build/app/outputs/flutter-apk/` |
