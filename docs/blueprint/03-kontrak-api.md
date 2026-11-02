# Blueprint 03 — Kontrak API

> **Ini satu-satunya antarmuka antara `api/` dan `mobile/`** karena keduanya repo terpisah.
> Kalau kontrak ini bocor atau berubah tanpa aturan, dua repo akan berbeda perilaku.
>
> Rujukan: `blueprint/01` (aturan M3) · `blueprint/02` (model data) · `docs/05` (Core)

Implementasi kontrak OpenAPI berada di `api/openapi/openapi.yaml`, dihasilkan
dari route Laravel dengan Scramble. Gunakan `api/bin/export-api-contract` untuk
menghasilkan ulang spesifikasi dan menyinkronkannya ke `mobile/contracts/`.
CI API memeriksa spesifikasi dengan skema migrasi lokal; notifikasi ke repo
mobile membawa OpenAPI terkompresi (gzip + Base64) melalui GitHub
`repository_dispatch`, lalu divalidasi sebelum disimpan. Mekanisme ini menjaga
kontrak tetap di bawah batas ukuran payload GitHub.

---

## 1. Prinsip

| # | Prinsip | Konsekuensi |
|---|---|---|
| 1 | **Setiap Action punya endpoint API** | UI web memakai session; endpoint API tetap tersedia & diuji (aturan M3) |
| 2 | **Satu bentuk respons untuk semua endpoint** | Klien Dart digenerate — bentuk harus seragam |
| 3 | **Endpoint tidak tahu kanal** | Token dan session menghasilkan pengguna yang sama |
| 4 | **Semua yang menyentuh uang idempoten** | Endpoint uang wajib menerima `Idempotency-Key` |
| 5 | **Kontrak digenerate dari kode, bukan ditulis manual** | Cegah dokumen dan kode berbeda |

---

## 2. Konvensi dasar

| Aspek | Ketentuan |
|---|---|
| Basis | `/api/v1` — versi di jalur |
| Autentikasi web | Session (web) — bukan API |
| Autentikasi API | `Authorization: Bearer <token>` (Sanctum) |
| Format data | **`snake_case`** — konsisten dengan database & idiomatik Dart |
| Uang | **Integer rupiah** (mis. `160545`) — Rupiah tidak memakai sen dalam praktik. Pembulatan hanya di tampilan |
| Waktu | **ISO 8601 UTC** (mis. `2026-10-04T07:30:00Z`) |
| Halaman | `?page=1&per_page=20` · maksimum `per_page` = 100 |
| Pengurutan | `?sort=-created_at` (tanda `-` = menurun) |
| Penyaringan | Parameter eksplisit per endpoint — **bukan** filter bebas dari klien |
| Bahasa | Pesan error berbahasa Indonesia; kode error berbahasa Inggris |

---

## 3. Bentuk respons

### Sukses

```json
{ "data": { "code": "MW-20261004-0001", "status": "paid", "total_amount": 108000 } }
```

### Sukses berhalaman

```json
{
  "data": [ { "code": "..." } ],
  "meta": { "page": 1, "per_page": 20, "total": 134, "last_page": 7 }
}
```

### Galat — **satu bentuk, tanpa perkecualian**

```json
{
  "message": "Saldo mengendap tidak mencukupi untuk mengaktifkan COD.",
  "code": "COLLATERAL_INSUFFICIENT",
  "errors": { "schedule_id": ["Jaminan kurang Rp 24.360."] }
}
```

| Kode HTTP | Kapan | `code` contoh |
|---|---|---|
| 400 | Permintaan tidak masuk akal | `INVALID_REQUEST` |
| 401 | Belum terautentikasi | `UNAUTHENTICATED` |
| 403 | Tidak berhak atas aksi/resource | `FORBIDDEN` |
| 404 | Tidak ada | `NOT_FOUND` |
| 409 | Bentrok keadaan (kursi sudah diambil, hold sudah dilepas) | `SEAT_TAKEN`, `HOLD_ALREADY_RELEASED` |
| 422 | Validasi gagal | `VALIDATION_FAILED` |
| 429 | Terlalu banyak permintaan | `RATE_LIMITED` |
| 500 | Kesalahan server | `SERVER_ERROR` |

> **Perbaikan dari sistem lama:** dulu galat validasi memakai kunci `data`, sedangkan klien membaca `errors` — akibatnya pesan asli tertelan dan berubah jadi galat generik. Kini **selalu** `message` + `errors`.

---

## 4. Idempotency

| Aturan | Ketentuan |
|---|---|
| Kapan wajib | Semua `POST` yang **mengubah uang** — pembayaran, pencairan, settlement, refund, top-up jaminan |
| Caranya | Header `Idempotency-Key: <uuid>` dari klien |
| Perilaku | Kunci sama dan payload sama → tidak membuat efek samping kedua; status resource dikembalikan. Kunci sama dengan payload berbeda → `409` |
| Masa simpan | Minimal 24 jam |
| Kasus khusus | Tanpa header pada endpoint uang → **ditolak** `400 IDEMPOTENCY_KEY_REQUIRED` |

> Ini yang mencegah masalah sistem lama: callback pencairan tanpa verifikasi menyebabkan **pengembalian dana berulang**.

---

## 5. Webhook masuk

| Aspek | Ketentuan |
|---|---|
| Jalur | `POST /api/v1/webhooks/{provider}` |
| Autentikasi | **Signature wajib** — bukan token pengguna |
| Signature tidak valid | `401` + **tidak diproses**; hanya referensi order dicatat pada log, bukan signature/payload |
| Duplikat | Wajib aman — event valid dideduplikasi dengan hash payload terpilih dan posting ledger idempoten |
| Balasan | Event valid yang sudah pernah diproses → `200`; refund parsial/chargeback atau reversal yang tidak dapat dibukukan → `409` untuk rekonsiliasi manual |
| Penyimpanan | Simpan metadata event dan hash; payload mentah tidak disimpan |

**Wajib signature:** pembayaran · pencairan · settlement · top-up. Di sistem lama, **hanya pencairan yang tidak diverifikasi** — dan justru itu yang berbahaya.

---

## 6. Daftar endpoint

Legenda: 🔑 = butuh `Idempotency-Key` · 🪪 = peran yang berhak

### Core — Identitas & Akses

| Metode | Jalur | 🪪 |
|---|---|---|
| POST | `/auth/register` | publik |
| POST | `/auth/login` | publik |
| POST | `/auth/logout` | semua |
| GET | `/me` · PATCH `/me` | semua |
| GET | `/me/memberships` | semua — daftar usaha & peran |
| POST | `/businesses` · GET `/businesses` · GET/PATCH `/businesses/{id}` | pemilik, admin |
| GET | `/businesses/{id}/members` | anggota berizin, admin |
| POST | `/businesses/{id}/invitations` | pemilik, admin |
| PATCH | `/businesses/{id}/members/{membership_id}` | pemilik, admin |
| DELETE | `/businesses/{id}/members/{user_id}` | pemilik, admin |
| POST | `/invitations/accept` | publik atau akun penerima |
| GET | `/permissions` | admin |

Undangan berlaku tujuh hari, token acaknya disimpan sebagai hash, dan token hanya
diberikan saat undangan dibuat. Tautan web membawa token pada URL fragment agar
tidak ikut terkirim dalam request GET; akun yang sudah ada harus masuk dengan
nomor telepon penerima. Sesi web tersedia pada `/login` dan `/register`.

**Satu akun satu fungsi.** `POST /auth/register` menerima `account_type` opsional dengan
nilai `customer` · `warung_owner` · `grosir_owner` · `agency_owner` (bawaan `customer`).
Fungsi akun menentukan area yang boleh dibuka (`customer`, `retail`, `grosir`, `agency`,
`driver`, `ops`, `admin` — lihat `blueprint/04` §1a). Nilai staf (`warung_guard`,
`grosir_staff`, `agency_staff`, `driver`) tidak boleh didaftarkan sendiri; akun itu lahir
dari penerimaan undangan usaha, dan undangan ditolak bila nomor tujuan sudah dipakai
fungsi lain. `POST /businesses` juga dibatasi: tipe usaha wajib sama dengan fungsi akun
pemiliknya, sehingga akun pelanggan menerima `422`.

Operator menetapkan admin platform awal melalui
`php artisan app:assign-platform-admin <nomor-telepon>` setelah akun dibuat.

`GET /me`, `POST /auth/login`, dan `POST /auth/register` mengembalikan `account_type` di
dalam objek `user` — klien memakai satu kolom itu untuk memilih beranda dan menu, sama
seperti web memakai `AccountType::areas()`. Daftar area tidak dikirim terpisah supaya
kontrak tidak punya dua sumber kebenaran.

### Core — Ledger, Dompet & Pencairan

| Metode | Jalur | 🪪 |
|---|---|---|
| POST 🔑 | `/businesses/{id}/settlements` | admin platform — settlement agency/grosir |
| GET | `/wallet/accounts` | pemilik |
| GET | `/wallet/transactions` | pemilik |
| GET | `/wallet/holds` | pemilik |
| POST 🔑 | `/wallet/collateral/topup` | pemilik |
| POST 🔑 | `/wallet/collateral/transfer-in` | pemilik |
| POST 🔑 | `/wallet/collateral/transfer-out` | pemilik |
| POST 🔑 | `/wallet/payment-methods` | pemilik — daftarkan sumber dana |

Kunci idempotensi untuk settlement dikirim melalui header `Idempotency-Key`.
Settlement mendebit `BUSINESS.PAYABLE:{id}` lalu mengkredit
`BUSINESS.AVAILABLE:{id}` untuk usaha yang sama; saldo payable usaha lain tidak
dapat dipakai.
| GET/POST 🔑 | `/withdrawals` | pemilik |
| GET | `/withdrawals/{code}` | pemilik |
| GET | `/settlements` · `/settlements/{code}` | pemilik |

### Core — Pembayaran & Notifikasi

| Metode | Jalur | 🪪 |
|---|---|---|
| POST 🔑 | `/payments/initiate` | pemilik agency — top-up jaminan melalui Midtrans Snap |
| GET | `/payments/{payment}` | pemilik pembayaran |
| POST | `/webhooks/midtrans` | publik + signature Midtrans |
| POST 🔑 | `/withdrawals` | pemilik agency — mengajukan pencairan |
| GET | `/withdrawals/{withdrawal}` | pemilik pengajuan — rekening hanya termasker |
| GET | `/admin/withdrawals/{withdrawal}` | admin — melihat rekening lengkap untuk transfer manual |
| POST | `/admin/withdrawals/{withdrawal}/approve` | admin — pengajuan tidak dapat disetujui pengajunya sendiri |
| POST | `/admin/withdrawals/{withdrawal}/complete` | admin — mencatat referensi transfer yang sudah dilakukan |
| POST | `/admin/withdrawals/{withdrawal}/fail` | admin — membalik reservasi bila transfer gagal |
| GET | `/notifications` · POST `/notifications/{notification}/read` | pemilik notifikasi |

### Konsol operasional (paritas layar `/admin`)

| Metode | Jalur | 🪪 |
|---|---|---|
| GET | `/admin/overview` | `ops.view` — antrean + kesehatan ledger |
| GET | `/admin/withdrawals?status=` | `withdrawal.review` — daftar pencairan |
| GET | `/admin/partners?type=` | `business.verify` — daftar mitra + status verifikasi |
| POST | `/admin/partners/{business}/verify` | `business.verify` — body `verified` (bool) |
| GET | `/admin/reconciliation` | `settlement.execute` — verifikasi ulang saldo dari entri |
| GET | `/admin/users?q=` | `permissions.read` — pengguna + peran platform |
| GET | `/admin/parameters` | `ops.view` — fee, PPN, timeout, minimum pencairan |
| GET | `/admin/support/tickets?status=` | `ops.view` — antrean dukungan seluruh pengguna |

### Dukungan & pelacakan (lintas peran)

| Metode | Jalur | 🪪 |
|---|---|---|
| GET | `/support/tickets?status=` | tiket milik sendiri |
| POST | `/support/tickets` | `category`, `subject`, `body`, `priority`, `booking_code?` |
| GET | `/support/tickets/{ticket}` | pemilik tiket atau staf — catatan internal hanya untuk staf |
| POST | `/support/tickets/{ticket}/messages` | `body`, `internal?` (internal hanya staf) |
| POST | `/support/tickets/{ticket}/close` | pemilik atau staf |
| GET | `/mad-trans/bookings/{booking}/tracking` | pemilik booking — status jadwal, armada/supir, penumpang naik, linimasa |

### Verifikasi dua langkah (MFA)

| Metode | Jalur | Catatan |
|---|---|---|
| POST | `/auth/login` | Bila MFA aktif: `data.mfa_required = true` + `data.mfa_token` (tanpa token akses) |
| POST | `/auth/mfa/verify` | `mfa_token`, `code` (6 angka), `device_name?` → token akses |
| GET | `/me/security` | status MFA + email tujuan kode |
| POST | `/me/security/two-factor` | kirim kode aktivasi → `mfa_token`, `expires_at` |
| POST | `/me/security/two-factor/confirm` | `code` → MFA aktif |
| DELETE | `/me/security/two-factor` | `password` → MFA dimatikan |

### Mad Warung — Pemilik

| Metode | Jalur |
|---|---|
| GET | `/warung/orders?business_id={id}` · `/warung/orders/{order}` |
| POST | `/warung/orders/{order}/decide` — setujui atau tolak |
| POST 🔑 | `/warung/orders/{order}/pay` — checkout Midtrans oleh pemilik |
| GET/PUT | `/warung/businesses/{business}/member-limits` · `/members/{member}/limit` |
| GET/POST | `/warung/businesses/{business}/shifts` |
| GET | `/warung/reports?business_id={id}` |

Penjaga membuat pesanan; pemilik warung menyetujui dan melakukan checkout
Midtrans Snap atas nama warung. Snap tidak menggunakan metode pembayaran
tersimpan milik pemilik.

### Mad Warung — Penjaga

| Metode | Jalur | 🪪 |
|---|---|---|
| GET | `/warung/wholesalers?business_id={id}` · `/warung/catalog?business_id={id}&wholesaler_id={id}` |
| POST 🔑 | `/warung/orders` |
| GET | `/warung/orders?business_id={id}` · `/warung/orders/{order}` |
| POST | `/warung/orders/{order}/receive` |
| GET | `/warung/businesses/{business}/shifts` |
| POST | `/warung/shifts/{shift}/clock-in` · `/clock-out` |

Penjaga tidak melakukan checkout pembayaran grosir; akses checkout hanya untuk
pemilik warung.

### Mad Warung — Grosir

| Metode | Jalur |
|---|---|
| PUT | `/warung/grosir/{wholesaler}/products/{product}/price` |
| GET | `/warung/orders?wholesaler_id={id}` · `/warung/orders/{order}` |
| POST | `/warung/orders/{order}/fulfill` — `prepare`, `ready`, atau `dispatch` |
| GET | `/warung/reports?business_id={id}` |

Produk global diterbitkan admin melalui `POST /admin/warung/products`.
Pembayaran pesanan mengkredit payable grosir; pencairan/settlement tetap
mengikuti alur ledger yang berizin.

### Mad Trans — Customer

| Metode | Jalur | 🪪 |
|---|---|---|
| GET | `/mad-trans/schedules?origin_city=&destination_city=&departure_date=&passengers=` | pengguna aktif |
| GET | `/mad-trans/schedules/{schedule}/seats?origin_stop_id=&destination_stop_id=` | pengguna aktif — kursi terisi per segmen + `has_driver_seat` |
| POST 🔑 | `/mad-trans/schedules/{schedule}/pickup-quote` | pengguna aktif — estimasi biaya jemput (lihat di bawah) |
| POST 🔑 | `/mad-trans/bookings` | pemesan; `payment_method=midtrans` atau `cod`; menerima `pickup_point` & `pickup_address` |
| POST 🔑 | `/mad-trans/bookings/{booking}/pay` | pemilik booking — checkout Midtrans |
| GET | `/mad-trans/bookings` · `/mad-trans/bookings/{booking}` | pemilik booking |
| POST | `/mad-trans/bookings/{booking}/cancel` | pemilik booking — pending online atau COD sebelum keberangkatan |

**Titik jemput (door to door).** `pickup_point` menerima koordinat `"lat, lng"` atau
tautan peta (`#map=z/lat/lng`, `?mlat=..&mlon=..` OpenStreetMap, `@lat,lng` atau
`?q=lat,lng` Google Maps). `pickup_address` adalah patokan alamat bebas. Keduanya
opsional; bila kosong perilakunya sama seperti booking biasa.

```json
// POST /mad-trans/schedules/{schedule}/pickup-quote
// { "origin_stop_id": 12, "pickup_point": "-6.28, 106.85" }
{
  "data": {
    "distance_km": 11.1,
    "free_radius_km": 5,
    "billable_km": 7,
    "fee": 28000,
    "fee_formatted": "Rp 28.000",
    "explanation": "Jarak 11,1 km, gratis 5 km, 7 km ditagih × Rp 4.000. Total Rp 28.000 ditambahkan ke pembayaran dan masuk ke pendapatan agen (bukan fee platform atau PPN)."
  }
}
```

Balasan `TravelBookingResource` menambahkan blok `pickup`
(`address`, `latitude`, `longitude`, `distance_km`, `fee`) dan `pickup_fee` di
samping `base_price`.

### Mad Trans — Agency (pemilik travel/rental)

| Metode | Jalur |
|---|---|
| GET/POST | `/mad-trans/businesses/{business}/vehicles` — `seat_rows` (array 1–8 baris, 0–4 kursi) **atau** `seat_layout` lama, plus `has_driver_seat` |
| GET/POST | `/mad-trans/businesses/{business}/routes` |
| GET/POST | `/mad-trans/businesses/{business}/schedules` |
| POST | `/mad-trans/schedules/{schedule}/approve` · `/enable-cod` |
| POST | `/mad-trans/schedules/{schedule}/start` · `/finish` — pemilik atau driver terpilih |
| GET | `/mad-trans/businesses/{business}/wallet` — saldo tersedia/jaminan/kewajiban · mutasi · hold aktif |
| GET | `/mad-trans/businesses/{business}/bookings?status=` — pesanan masuk ke jadwal agency |
| GET | `/mad-trans/businesses/{business}/withdrawals` — riwayat pencairan (pengajuan lewat `POST /withdrawals`) |
| GET | `/mad-trans/businesses/{business}/rental-bookings` — pesanan sewa yang dikelola agency |

### Mad Trans — Driver

| Metode | Jalur |
|---|---|
| POST | `/mad-trans/bookings/{booking}/passengers/{passenger}/board` |
| POST | `/mad-trans/bookings/{booking}/passengers/{passenger}/no-show` |

Hold COD dihitung dan dibuat per penumpang saat booking dibuat; booking ditolak
jika saldo jaminan bebas tidak mencukupi. Jadwal dapat mengaktifkan COD tanpa
menahan saldo untuk kursi yang belum dipesan. Boarding menagihkan service fee,
platform fee, dan PPN ke ledger secara idempoten; pembatalan sebelum berangkat
dan no-show melepaskan hold tanpa mencatat pendapatan.
Penyelesaian jadwal melepaskan hold COD yang tersisa dan menandai penumpang COD
yang belum boarding sebagai no-show; penutupan ditolak bila masih ada pembayaran
online yang sedang diproses.

---

## 6b. Padanan API untuk fitur web (Fase 9)

Fitur yang sebelumnya hanya ada di halaman web kini punya endpoint sendiri supaya
aplikasi mobile tidak perlu menyalin aturan bisnis. Semuanya memanggil kelas Core
yang sama dengan web (`CreateRentalBooking`, `InitiateRentalPayment`, `ResolvePromo`,
`LoyaltySummary`, `ReferralCode`) — tidak ada duplikasi logika.

### Rental — Customer

| Metode | Jalur | 🪪 |
|---|---|---|
| GET | `/mad-trans/businesses/{business}/rental-vehicles` | pengguna aktif — daftar kendaraan + supir + `self_drive_ready` |
| POST 🔑 | `/mad-trans/businesses/{business}/rental-bookings` | `booking.create` — body `rental_type`, `vehicle_id`, `driver_membership_id`, `start_at`, `end_at`, `promo_code`, `idempotency_key` |
| GET | `/mad-trans/rental-bookings` · `/mad-trans/rental-bookings/{rental}` | penyewa |
| POST | `/mad-trans/rental-bookings/{rental}/pay` | penyewa — Snap token; idempoten via `idempotency_key` |

`rental_type` menolak `self_drive` dengan `422 RENTAL_REJECTED` bila dokumen belum
terverifikasi (aturan yang sama seperti web). Respons `pay` selalu `200` agar
bentuknya bisa diketik di kontrak; pemanggilan ulang mengembalikan intent yang sama.

### Dokumen lepas kunci

| Metode | Jalur | 🪪 |
|---|---|---|
| GET | `/rental-documents` | pengguna aktif — status `ktp`, `sim`, `selfie` + `verified_all` |
| POST | `/rental-documents` | multipart `type` + `file` (jpg/jpeg/png/pdf, maks 4 MB) |

Unggah ulang mengembalikan status ke `uploaded` dan mengosongkan verifikasi —
sama seperti web. Verifikasi tetap hanya lewat admin platform.

### Promo, referral, level

| Metode | Jalur | Keterangan |
|---|---|---|
| POST | `/promos/validate` | `code`, `applies_to` (`travel`/`rental`), `business_id`, `base_amount` → diskon + `cost_bearer`; gagal `422 PROMO_REJECTED` |
| GET | `/referral` | kode milik pengguna, `share_url`, daftar undangan, promo hadiah |
| GET | `/loyalty` | poin, level, level berikutnya, misi bulan ini |

---

## 7. Cara menghasilkan klien Dart

Alur yang dipakai sekarang (Fase 9):

```
1. bash api/bin/export-api-contract      # Scramble → api/openapi/openapi.yaml
                                         # + salinan ke mobile/contracts/openapi.yaml
2. bash api/bin/generate-dart-client     # openapi-generator (dart-dio) → mobile/api_client
3. aplikasi mobile memakai paket lokal:
     dependencies:
       gomad_api:
         path: api_client
```

Skrip generator memilih backend yang tersedia: `openapi-generator-cli` lokal →
image Docker `openapitools/openapi-generator-cli` → `npx @openapitools/openapi-generator-cli`.
Setelah membuat klien ia menjalankan `dart pub get` bila Dart SDK ada.

Pipeline CI yang diidealkan:

```
1. Kontrak digenerate dari kode (OpenAPI) saat CI berjalan
2. Disimpan sebagai artefak + di-commit ke repo `mobile/`
3. `mobile/` menjalankan generator → klien Dart
4. Repo `mobile/` dibuatkan PR otomatis bila kontrak berubah
```

**Aturan:** klien Dart **tidak boleh ditulis manual**. Kalau ditulis manual, dua repo akan berbeda perilaku tanpa ada yang menyadari — dan itu risiko terbesar dari pilihan dua repo.

**Catatan generator:** kontrak memakai OpenAPI 3.1; generator `dart-dio` masih
menandainya "in development" tetapi menghasilkan 22 kelompok API dan ~155 model
yang bisa dikompilasi. Skrip generator juga:

1. menaikkan batas SDK paket hasil generate ke `>=3.8.0` (dibutuhkan json_serializable modern),
2. menjalankan `dart run build_runner build` untuk membuat `*.g.dart`,
3. menghapus hasil generate sebelumnya (generator tidak menghapus model usang),
4. menjalankan `dart analyze` dan berhenti bila masih ada `error`.

Bentuk respons yang statusnya dinamis (mis. `201` vs `200` pada satu method) jatuh
menjadi `string` — karena itu endpoint uang di API mobile memakai satu status tetap.
Skema kosong (`{}`) juga menghasilkan kelas Dart tanpa properti; daftar kerjanya:
`bash bin/check-openapi-schemas`.

---

## 8. Versi & kompatibilitas

| Aturan | Ketentuan |
|---|---|
| Perubahan aman | Menambah field, menambah endpoint — boleh langsung di `v1` |
| Perubahan merusak | Menghapus/merename field, mengubah tipe — **wajib** `v2` |
| Aplikasi terpasang | Tidak bisa dipaksa memperbarui — jadi `v1` dan `v2` hidup berdampingan |
| Penghapusan versi | Hanya setelah pemakaian `v1` mendekati nol (diukur, bukan dikira) |
| Penanda usang | Respons menyertakan header `Deprecation` bila endpoint akan dihapus |

---

## 9. Status keputusan

### Sudah diputuskan (2026-10-04)

| # | Item | Keputusan |
|---|---|---|
| 1 | **Asumsi PPN** | ✅ **Ditambahkan ke pembeli** → bentuk `ppn_amount` di respons harga jadi jelas |
| 2 | **Rancangan fallback kartu** | ✅ Metode default (token) + **fallback QRIS/VA** → `/payments/initiate` mengembalikan metode utama, dan menyediakan jalur QRIS/VA bila gagal |
| 3 | **Pemelihara katalog grosir** | ✅ **Hibrida** → `POST /grosir/products` = GoMad (saat onboarding) · `PATCH /grosir/prices` = grosir |

### Masih menunggu

| # | Menunggu | Memengaruhi |
|---|---|---|
| 1 | **Wilayah pertama** dari 3 wilayah sasaran | Bentuk data rute & grosir terdekat |
| 2 | **Prefix kode referensi** per vertical | Format `code` di respons |
