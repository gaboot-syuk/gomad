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
│   ├── main.dart        # kerangka aplikasi + pemilihan halaman
│   └── src/
│       ├── api_config.dart   # base URL + Dio + klien
│       ├── session.dart      # token, refresh otomatis, retry 401
│       ├── token_store.dart  # penyimpanan token (memori → aman di produksi)
│       ├── format.dart       # Rupiah, jam, tanggal
│       └── ui/               # halaman: login, home, seat, payment
└── tool/pilot_smoke.dart     # uji alur pilot langsung ke server
```

---

## 4. Alur pilot (Android)

```
Login → Cari jadwal → Pilih kursi + data penumpang → Pesan → Bayar (Snap) → Batalkan (demo)
```

| Langkah | Endpoint | Bukti |
|---|---|---|
| Login | `POST /v1/auth/login` | token 30 hari, nama perangkat dicatat |
| Profil | `GET /v1/me` | nama pengguna tampil di AppBar |
| Perpanjang sesi | `POST /v1/auth/refresh` | token berotasi, token lama dihapus |
| Cari jadwal | `GET /v1/mad-trans/schedules` | menerima ISO 8601 dari klien |
| Peta kursi | `GET /v1/mad-trans/schedules/{id}/seats` | kursi terisi tidak bisa dipilih |
| Pesan | `POST /v1/mad-trans/bookings` | `Idempotency-Key` per percobaan |
| Bayar | `POST /v1/mad-trans/bookings/{id}/pay` | Snap token + redirect URL |

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
| Kerangka proyek `android/` | Dibuat sekali dengan `flutter create .` di mesin ber-SDK lengkap |

---

## 6. Status kriteria Fase 9

| Kriteria | Status | Bukti |
|---|---|---|
| Klien Dart otomatis dari kontrak | ✅ | `bin/generate-dart-client` (22 kelompok API, ±155 model, `dart analyze` tanpa error) |
| Auth token & refresh bekerja | ✅ | `test/session_test.dart` (4 tes) + `tool/pilot_smoke.dart` ke server lokal |
| Satu alur pilot jalan di Android | 🟡 | Alur lengkap terbukti lewat klien dari host; instalasi di perangkat butuh `flutter create .` |
| Tidak ada logika bisnis diduplikasi | ✅ | Semua perhitungan di Core Laravel; aplikasi hanya memformat tampilan |
