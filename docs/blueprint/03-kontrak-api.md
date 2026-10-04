# Blueprint 03 — Kontrak API

> **Ini satu-satunya antarmuka antara `api/` dan `mobile/`** karena keduanya repo terpisah.
> Kalau kontrak ini bocor atau berubah tanpa aturan, dua repo akan berbeda perilaku.
>
> Rujukan: `blueprint/01` (aturan M3) · `blueprint/02` (model data) · `docs/05` (Core)

Implementasi kontrak OpenAPI berada di `api/openapi/openapi.yaml`, dihasilkan
dari route Laravel dengan Scramble. Gunakan `api/bin/export-api-contract` untuk
menghasilkan ulang spesifikasi dan menyinkronkannya ke `mobile/contracts/`.
CI API memeriksa spesifikasi tetap mutakhir; notifikasi ke repo mobile memakai
GitHub `repository_dispatch` dan baru aktif setelah variabel repo serta token
disiapkan.

---

## 1. Prinsip

| # | Prinsip | Konsekuensi |
|---|---|---|
| 1 | **Setiap Action punya endpoint API** | Web memakai Inertia, tapi endpoint tetap ada & diuji (aturan M3) |
| 2 | **Satu bentuk respons untuk semua endpoint** | Klien Dart digenerate — bentuk harus seragam |
| 3 | **Endpoint tidak tahu kanal** | Token dan session menghasilkan pengguna yang sama |
| 4 | **Semua yang menyentuh uang idempoten** | Endpoint uang wajib menerima `Idempotency-Key` |
| 5 | **Kontrak digenerate dari kode, bukan ditulis manual** | Cegah dokumen dan kode berbeda |

---

## 2. Konvensi dasar

| Aspek | Ketentuan |
|---|---|
| Basis | `/api/v1` — versi di jalur |
| Autentikasi web | Session (Inertia) — bukan API |
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
| Perilaku | Kunci sama dikirim ulang → **respons pertama dikembalikan apa adanya**, tanpa efek samping |
| Masa simpan | Minimal 24 jam |
| Kasus khusus | Tanpa header pada endpoint uang → **ditolak** `400 IDEMPOTENCY_KEY_REQUIRED` |

> Ini yang mencegah masalah sistem lama: callback pencairan tanpa verifikasi menyebabkan **pengembalian dana berulang**.

---

## 5. Webhook masuk

| Aspek | Ketentuan |
|---|---|
| Jalur | `POST /api/v1/webhooks/{provider}` |
| Autentikasi | **Signature wajib** — bukan token pengguna |
| Signature tidak valid | `401` + **tidak diproses** + dicatat sebagai percobaan mencurigakan |
| Duplikat | Wajib aman — diperiksa lewat `idempotency_key` di ledger |
| Balasan | Selalu `200` bila sudah pernah diterima, agar pengirim berhenti mengulang |
| Penyimpanan | Simpan payload mentah untuk audit & rekonsiliasi |

**Wajib signature:** pembayaran · pencairan · settlement · top-up. Di sistem lama, **hanya pencairan yang tidak diverifikasi** — dan justru itu yang berbahaya.

---

## 6. Daftar endpoint

Legenda: 🔑 = butuh `Idempotency-Key` · 🪪 = peran yang berhak

### Core — Identitas & Akses

| Metode | Jalur | 🪪 |
|---|---|---|
| POST | `/auth/login` | publik |
| POST | `/auth/logout` | semua |
| GET | `/me` · PATCH `/me` | semua |
| GET | `/me/memberships` | semua — daftar usaha & peran |
| POST | `/businesses` · GET/PATCH `/businesses/{id}` | pemilik, admin |
| GET/POST | `/businesses/{id}/members` | pemilik, admin |
| PATCH/DELETE | `/businesses/{id}/members/{user_id}` | pemilik, admin |
| GET | `/permissions` | admin |

### Core — Ledger, Dompet & Pencairan

| Metode | Jalur | 🪪 |
|---|---|---|
| GET | `/wallet/accounts` | pemilik |
| GET | `/wallet/transactions` | pemilik |
| GET | `/wallet/holds` | pemilik |
| POST 🔑 | `/wallet/collateral/topup` | pemilik |
| POST 🔑 | `/wallet/collateral/transfer-in` | pemilik |
| POST 🔑 | `/wallet/collateral/transfer-out` | pemilik |
| POST 🔑 | `/wallet/payment-methods` | pemilik — daftarkan sumber dana |
| GET/POST 🔑 | `/withdrawals` | pemilik |
| GET | `/withdrawals/{code}` | pemilik |
| GET | `/settlements` · `/settlements/{code}` | pemilik |

### Core — Pembayaran & Notifikasi

| Metode | Jalur | 🪪 |
|---|---|---|
| POST 🔑 | `/payments/initiate` | semua pihak yang membayar |
| GET | `/payments/{code}` | pemilik pesanan |
| POST | `/webhooks/{provider}` | publik + signature |
| GET | `/notifications` · POST `/notifications/{id}/read` | semua |
| POST | `/devices` | semua — daftar token push mobile |

### Mad Warung — Pemilik

| Metode | Jalur |
|---|---|
| GET | `/mad-warung/outlets` |
| GET/PUT | `/mad-warung/limits` · `/mad-warung/limits/{user_id}` |
| GET/POST/PATCH | `/mad-warung/shifts` |
| GET | `/mad-warung/orders` · `/mad-warung/orders/{code}` |
| GET | `/mad-warung/approvals` · POST `/mad-warung/approvals/{code}/decide` |
| GET | `/mad-warung/reports` |

### Mad Warung — Penjaga

| Metode | Jalur | 🪪 |
|---|---|---|
| GET | `/mad-warung/catalog?grosir_id=` |
| POST 🔑 | `/mad-warung/orders` |
| POST 🔑 | `/mad-warung/orders/{code}/pay` |
| GET | `/mad-warung/orders` · `/mad-warung/orders/{code}` |
| POST | `/mad-warung/orders/{code}/confirm-received` |

### Mad Warung — Grosir

| Metode | Jalur |
|---|---|
| GET/POST/PATCH | `/mad-warung/grosir/products` · `/prices` |
| GET | `/mad-warung/grosir/orders` · `/orders/{code}` |
| POST | `/mad-warung/grosir/orders/{code}/ready` |
| GET | `/mad-warung/grosir/settlements` |

### Mad Trans — Customer

| Metode | Jalur | 🪪 |
|---|---|---|
| GET | `/mad-trans/routes` · `/mad-trans/schedules` | — |
| GET | `/mad-trans/schedules/{id}/seats` | — |
| POST 🔑 | `/mad-trans/bookings` | ya |
| POST 🔑 | `/mad-trans/bookings/{code}/pay` | ya |
| GET | `/mad-trans/bookings` · `/{code}` | — |
| POST | `/mad-trans/bookings/{code}/cancel` | — |
| GET | `/mad-trans/rental-vehicles` | — |
| POST 🔑 | `/mad-trans/rentals` · POST `/rentals/{code}/documents` | ya |

### Mad Trans — Agency (pemilik travel/rental)

| Metode | Jalur |
|---|---|
| GET/POST/PATCH | `/mad-trans/vehicles` (termasuk `seat_layout`) |
| GET/POST/PATCH | `/mad-trans/routes` · `/route-stops` · `/route-pricing` |
| GET/POST/PATCH | `/mad-trans/schedules` |
| POST | `/mad-trans/schedules/{id}/approve` · `/reject` |
| GET | `/mad-trans/agency/bookings` · `/rentals` |
| POST | `/mad-trans/transfers` (transfer penumpang) |

### Mad Trans — Driver

| Metode | Jalur |
|---|---|
| GET | `/mad-trans/driver/today` |
| POST | `/mad-trans/driver/schedules/{id}/start` · `/finish` |
| GET | `/mad-trans/driver/schedules/{id}/passengers` |
| POST | `/mad-trans/driver/bookings/{code}/confirm-cod` |

---

## 7. Cara menghasilkan klien Dart

```
1. Kontrak digenerate dari kode (OpenAPI) saat CI berjalan
2. Disimpan sebagai artefak + di-commit ke repo `mobile/`
3. `mobile/` menjalankan generator → klien Dart
4. Repo `mobile/` dibuatkan PR otomatis bila kontrak berubah
```

**Aturan:** klien Dart **tidak boleh ditulis manual**. Kalau ditulis manual, dua repo akan berbeda perilaku tanpa ada yang menyadari — dan itu risiko terbesar dari pilihan dua repo.

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
