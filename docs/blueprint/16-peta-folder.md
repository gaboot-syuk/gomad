# Blueprint 16 — Peta Folder

> Satu akun satu fungsi (lihat `blueprint/04` §1a) juga tercermin di susunan folder:
> satu area = satu folder controller + satu folder view + satu berkas rute.
> URL tidak berubah — pemisahan ini soal kejelasan kode, bukan alamat.

---

## 1. Controller — `app/Http/Controllers/Web/`

| Folder | Isi | Area |
|---|---|---|
| `MadWarung/` | `WarungController` (aksi POST pesanan, katalog, shift), `WarungPortalController` (semua halaman ruang kerja) | `retail` + `grosir` |
| `MadTrans/Customer/` | `MadTransController`, `RentalController`, `RentalCalendarController`, `RentalDocumentController`, `TrackingController` | `customer` |
| `MadTrans/Travel/` | `AgencyPortalController`, `AgencyTravelController`, `AgencyPromoController` | `agency` |
| `MadTrans/Driver/` | `DriverTravelController` | `driver` |
| `Platform/Admin/` | `AdminConsoleController`, `AdminPromoController`, `AdminDocumentController`, `AdminSupportController` | `ops` + `admin` |
| `Shared/` | `AuthController`, `MfaController`, `AccountController`, `NotificationController`, `SupportController`, `ReferralController`, `LoyaltyController`, `BusinessController`, `InvitationController`, `HomeController`, `PublicController` | bersama |

### API — `app/Http/Controllers/Api/V1/`

| Folder | Isi | Dipakai |
|---|---|---|
| `Shared/` | `AuthController`, `IdentityController`, `PermissionController`, `BusinessController`, `BusinessInvitationController`, `BusinessMemberController`, `NotificationController`, `SupportController`, `EngagementController`, `PublicController` | semua akun |
| `MadWarung/` | `WarungCatalogController`, `WarungMemberLimitController`, `WarungOrderController`, `WarungReportController`, `WarungShiftController` | retail + grosir |
| `MadTrans/Customer/` | `MadTransController` | pelanggan + supir (boarding) |
| `MadTrans/Travel/` | `MadTransAgencyController` | agency travel |
| `MadTrans/Rental/` | `RentalController`, `RentalDocumentController` | pelanggan & agency rental |
| `Platform/` | `AdminConsoleController` | ops + admin |
| `Money/` | `PaymentController`, `PaymentWebhookController`, `WithdrawalController`, `SettlementController` | pembeli, mitra, platform |

---

## 2. View — `resources/views/`

```
public/               halaman pengunjung tanpa akun (beranda, cari, warung publik, mitra, unduh)
madwarung/
  index.blade.php     pilih outlet
  _nav.blade.php      tab ruang kerja (disaring izin)
  dashboard|catalog|orders|report.blade.php   halaman bersama retail & grosir
  retail/             halaman khusus warung: guards (penjaga & plafon), shifts
  grosir/             disiapkan untuk halaman yang benar-benar berbeda dari retail
madtrans/
  customer/           beranda travel, pilih kursi, pesanan, e-ticket, batal, pelacakan
  travel/             ruang kerja agency: armada, rute, jadwal, pengemudi, pesanan, promo, pelacakan
  rental/             rental: meja rental agency, kalender, dokumen penyewa, sewa pelanggan
  driver/             tugas supir
platform/
  admin/              konsol platform (dasbor, pencairan, settlement, rekonsiliasi, mitra, pengguna, parameter, promo, dokumen, dukungan)
  ops/                disiapkan untuk halaman yang hanya ops (belum ada)
shared/
  home/               beranda pelanggan
  account/ auth/ support/ notifications/ referral/ loyalty/ businesses/ invitations/
components/gm/        komponen design system (sadar-tema, lihat blueprint/15)
layouts/app.blade.php kepala dokumen + pemilih tema (`data-theme`)
layouts/partials/
  operational-shell.blade.php   kerangka Classic Minimalist (sidebar + top bar)
  consumer-shell.blade.php      kerangka Glassmorphism (navigasi mengambang + bawah)
  flash.blade.php               galat validasi, status, galat umum
```

**Kerangka tema.** Hanya ada **satu** `layouts/app.blade.php`; ia memilih kerangka
dari `App\Support\DesignTheme`. Semua 66 view tetap `@extends('layouts.app')`, jadi
tema berlaku menyeluruh tanpa menyentuh view satu per satu — dan mustahil ada halaman
operasional yang diam-diam memakai kaca.

**Catatan jujur soal `madwarung/`.** Halaman `dashboard`, `catalog`, `orders`, dan `report`
memang dipakai retail **dan** grosir; perbedaannya (istilah, aksi, blok tambahan) masih
ditangani satu variabel `$isGrosir` di dalam berkas. Memisahkannya menjadi dua berkas
berarti menggandakan ±500 baris markup yang mudah melenceng — jadi yang dipisah lebih dulu
adalah halaman yang benar-benar berbeda (`retail/guards`, `retail/shifts`). Saat pemisahan
berikutnya dilakukan, halaman grosir tinggal dibuat di `madwarung/grosir/` tanpa mengubah
nama rute. Penjaga warung memakai halaman retail yang sama; yang membatasi pekerjaannya
adalah izin (`member_limit.manage`, `shift.manage` vs `shift.self`), bukan folder lain.

---

## 3. Rute — `routes/`

| Berkas | Area | Middleware |
|---|---|---|
| `web.php` | publik, auth, area bersama, pendaftaran berkas area | `auth`, `area:…` |
| `areas/madwarung.php` | `/mad-warung/**` | `area:retail,grosir` |
| `areas/madtrans-customer.php` | `/mad-trans`, bookings, schedules, rental pelanggan | `area:customer` |
| `areas/madtrans-agency.php` | `/mad-trans/agency/**` | `area:agency` |
| `areas/madtrans-driver.php` | `/mad-trans/driver/**` | `area:driver` |
| `areas/platform.php` | `/admin/**` | `area:ops` |
| `api.php` | `/api/v1/**` | token Sanctum |
| `api/shared.php` | auth, identitas, izin, usaha, notifikasi, dukungan, promo/referral/level, publik, webhook, pembayaran & pencairan | `auth:sanctum` |
| `api/madwarung.php` | `/warung/**` | `auth:sanctum` + `area:retail,grosir` |
| `api/madtrans.php` | `/mad-trans/**`, `/rental-documents` | `area:customer` · `area:driver` · `area:agency` (+ `area:agency,driver` untuk mulai/tutup perjalanan) |
| `api/platform.php` | `/admin/**`, `/admin/withdrawals/**` | `auth:sanctum` + `area:ops` |

Unduhan dokumen rental (`/mad-trans/rental-documents/{document}/file`) dipakai pelanggan
**dan** agency, jadi berada di grup `area:customer,agency` di `web.php`.

---

## 4. Aturan penamaan

| Hal | Aturan |
|---|---|
| Namespace controller | mengikuti folder, `App\Http\Controllers\Web\{Area}\{Controller}` |
| Nama view | `{area}.{sub}.{halaman}` tanpa kata `mad-` (folder sudah menyebut vertical) |
| Nama rute | tetap `warung.*`, `mad-trans.*`, `admin.*` — klien mobile & tautan lama tidak berubah |
| Berkas rute area | satu berkas per area, di-`group(base_path(...))` dari `web.php` |
| Kelas domain bersama | `App\Core\{BoundedContext}\{Layer}` — `Domain` tidak boleh menyentuh HTTP |
| Kelas bantu presentasi | `App\Support\*` — `Navigation` (menu), `DesignTheme` (tema), `Rupiah` (format) |
