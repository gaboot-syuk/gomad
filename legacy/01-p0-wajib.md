# P0 — Wajib

> **Tanpa ini Tahap 1 tidak bisa jalan.**
> Enam artefak. Semuanya menyangkut "bentuk nyata sistem", bukan opini.

| # | Yang dibutuhkan | Kenapa |
|---|---|---|
| 1 | Skema database | Ini artefak paling berharga |
| 2 | Daftar status & lifecycle | Biasanya hanya hidup di kode |
| 3 | Aturan bisnis | Aturan tersembunyi = sumber bug saat rebuild |
| 4 | Daftar role & permission | Menentukan desain `Core/Access` |
| 5 | Daftar endpoint | Peta permukaan API lama vs baru |
| 6 | Stack & infrastruktur | Menentukan seberapa banyak yang bisa diselamatkan |

---

## P0-1 · Skema database

**Yang dibutuhkan:** DDL dump, file `migrations/`, atau ERD.

**Kenapa:** Ini artefak paling berharga. Dari sini kita tahu entity nyatanya, bukan tebakan.

**Format ideal** (urut dari paling baik):
1. `mysqldump --no-data --routines --triggers` dari **database produksi** → file `.sql`.
2. ERD (dbdiagram.io / drawSQL / MySQL Workbench) → `.png` + source `.dbml`.
3. Folder `migrations/` apa adanya.

**Cara cepat dapat:**
```bash
mysqldump -h <host> -u <user> -p --no-data --skip-comments <dbname> > schema.sql
```
Kalau tidak ada akses produksi, kirim hasil `SHOW CREATE TABLE <tabel>` untuk tabel inti saja.

**Di repo ini:** sudah ada — `gomadid/database/migrations/` berisi **68 file migration** (2016_08_03 s/d 2026_09_08). Tabel inti yang terbaca dari situ:

`users`, `agencies`, `agency_verifications`, `agency_policies`, `routes`, `route_stops`, `route_pricing`, `vehicles`, `schedules`, `schedule_stops`, `bookings`, `booking_passengers`, `payments`, `cash_payments`, `payment_agents`, `agency_wallets`, `wallet_transactions`, `withdrawals`, `settlements`, `reviews`, `promos`, `rentals`, `customer_documents`, `vehicle_rental_settings`, `notifications`, `driver_locations`, `passenger_transfers`, `user_devices`, `platform_settings`, plus `pos_*` (POS warung), `level_*` / `missions` / `xp_logs`, `conversations` / `chat_messages`, dan tabel wilayah Laravolt (`provinces`→`villages`).

**Yang masih kurang:**
- **Verifikasi drift.** Migration = niat historis, belum tentu sama dengan skema produksi (ada 4 migration "add missing indexes", "make column nullable", dll — tanda evolusi cepat). DDL dump produksi tetap wajib untuk jadi sumber kebenaran.
- **ERD.** Belum ada. Kalau perlu, bisa digenerate dari migrations.
- **Catatan kolom non-obvious**: `agency_wallets` punya 5 saldo (`available`, `pending`, `deposit`, `cod_hold_balance`, `total_earned`/`total_withdrawn`) — perlu penjelasan makna bisnis tiap kolom, bukan cuma tipenya.

---

## P0-2 · Daftar status & lifecycle

**Yang dibutuhkan:** semua nilai enum/status untuk **jadwal, booking, trip, pembayaran, settlement**.

**Kenapa:** Biasanya hanya hidup di kode, tidak pernah ditulis. Ini yang jadi *state machine* di Core baru.

**Format ideal:** per entitas — daftar nilai, **transisi legal** (dari → ke), **pemicu** transisi, dan **aktor** yang boleh memicu. Daftar nilai saja belum cukup.

**Di repo ini:** sebagian sudah ada di `gomadid/app/Enums/` — **10 file**:

| File | Entitas |
|---|---|
| `BookingStatus.php` | booking / trip |
| `PaymentStatus.php` | pembayaran |
| `RentalStatus.php` | rental |
| `SettlementStatus.php` | settlement warung |
| `WithdrawalStatus.php` | pencairan |
| `DocumentVerificationStatus.php` | verifikasi dokumen |
| `WalletTransactionType.php` | transaksi wallet |
| `UserRole.php` | role |
| `RentalType.php` | tipe rental |
| `TravelClass.php` | kelas travel |

Nilai yang terbaca: `BookingStatus` = `pending → confirmed → paid → on_going → completed` (+ `cancelled` dari pending/confirmed/paid). `PaymentStatus` = **12 nilai** yang menampung 3 jalur paralel: digital (`pending/paid/failed/refunded/expired`), COD (`cod_pending/cod_confirmed`), OTS (`ots_pending/ots_confirmed`), refund (`refund_pending/approved/rejected`). `RentalStatus` = `pending → paid → active → returned → completed`. `SettlementStatus` = `pending → paid → verified` (+ `overdue`). `WithdrawalStatus` = `pending → approved → processing → completed` (+ `rejected/failed`).

**Yang masih kurang (gap penting):**
- ⚠️ **Tidak ada `ScheduleStatus.php`.** Status jadwal kemungkinan diturunkan dari kolom tanggal + field `approval` (lihat migration `2026_08_25_000002_add_approval_to_schedules_table`), bukan enum. **Ini harus diklarifikasi** — kalau memang tidak ada, ini titik paling rawan saat rebuild.
- ⚠️ **Tidak ada `TripStatus`.** "Trip" kemungkinan == `BookingStatus` tahap `on_going` + `DriverLocation`. Perlu dikonfirmasi apakah trip itu entitas tersendiri atau bagian dari booking.
- ⚠️ **Transisi & pemicu belum terdokumentasi.** Yang ada hanya daftar nilai. Siapa yang boleh mengubah `pending → confirmed`? Apa yang men-trigger `expired`? Berapa lama? Ini yang harus ditulis manual.
- Status **rental pickup/return** (`verify pickup`, `verify return`) belum terlihat sebagai enum — perlu dikonfirmasi.

---

## P0-3 · Aturan bisnis

**Yang dibutuhkan:** komisi, TTL seat lock, cutoff penjualan, refund/cancel, transfer penumpang, harga per segmen (multi-stop), mekanik Koin COD.

**Kenapa:** Aturan tersembunyi = sumber bug saat rebuild.

**Format ideal:** tabel per aturan — nama, nilai, unit, sumber kebenaran (file:kode), apakah configurable, siapa yang mengubah.

**Di repo ini:** sebagian besar hidup di dua tempat:

1. **`gomadid/config/gomad.php`** — komisi platform & warung, service fee, limit penarikan, prefix kode booking (`GM`) & kode cash (`WM`), timeout pembayaran, jadwal settlement (Senin).
2. **`gomadid/app/Services/`** — **26 service**, 25 di antaranya berisi logika bisnis nyata:

   `BookingService`, `RentalService`, `WalletService`, `WithdrawalService`, `SettlementService`, `PaymentService`, `CashPaymentService`, `PaymentAgentService`, `PaymentMethodService`, `PricingService`, `PromoService`, `PassengerTransferService`, `ScheduleService`, `RouteService`, `DriverService`, `VerificationService`, `LevelService`, `OverloadService`, `ManualInputService`, `OnboardingService`, `AgencyDashboardService`, `AgencyProfileService`, `NotificationService`, `DeviceService`, `SitemapService`, `CloudinaryService`.

**Yang masih kurang / perlu diklarifikasi:**
- ⚠️ **TTL seat lock.** Dari kode terlihat pola `lockForUpdate` saat booking + timeout pembayaran, tapi **konsep "seat di-lock selama X menit" belum jelas apakah ada**. Ini perlu jawaban eksplisit: kalau tidak ada, Core baru bisa jadi *lebih* ketat dari sistem lama — dan itu keputusan produk, bukan teknis.
- ⚠️ **Cutoff penjualan** — belum ditemukan aturan eksplisit (mis. "tidak bisa pesan H-1 jam"). Perlu dikonfirmasi apakah ada atau tidak.
- ⚠️ **Koin COD** — ada `cod_hold_balance` di `agency_wallets` dan alur kode `WM` + PIN warung, tapi mekanik lengkapnya (kapan koin jadi saldo, apa yang terjadi kalau warung tutup) perlu ditulis manual.
- **Refund/cancel** — aturan ada di kode, tapi kebijakan (siapa menanggung, potongan berapa) harus ditulis manusia.

---

## P0-4 · Daftar role & permission

**Yang dibutuhkan:** customer, agency owner, staf agency, driver, admin, finance — siapa boleh apa.

**Kenapa:** Menentukan desain `Core/Access`.

**Di repo ini:** `gomadid/app/Enums/UserRole.php` mendefinisikan **5 role**:
`customer`, `agency`, `driver`, `admin`, `payment_agent`.

Middleware terpisah per role di `app/Http/Middleware/Api/` dan `.../Web/`.

**Yang masih kurang (gap penting):**
- ⚠️ **"Staf agency" tidak ada.** Sekarang agency = 1 user. Kalau bisnis butuh multi-user per agency (operator, kasir, dispatcher), ini **fitur baru**, bukan migrasi — dan harus masuk desain `Core/Access` dari awal.
- ⚠️ **"Finance" tidak ada.** Peran keuangan saat ini melekat di `admin`. Kalau akan dipisah, perlu daftar aksi keuangan mana yang boleh dilakukan siapa.
- ⚠️ **Tidak ada tabel permission.** Semua kontrol berbasis role tunggal + middleware + pengecekan di service/controller. Untuk Core baru, daftar *aksi* (bukan hanya role) perlu diturunkan manual dari kode.
- Yang sudah jelas dan bisa dikirim: **5 role + matriks role × menu web** (dari struktur `resources/views/` yang mirror per role).

---

## P0-5 · Daftar endpoint

**Yang dibutuhkan:** cukup **method + path + siapa yang akses**. Tidak perlu spec lengkap.

**Kenapa:** Peta permukaan API lama vs baru.

**Di repo ini:** sudah ada secara mentah:
- `gomadid/routes/api.php` — API per role, prefix `/api/v1`, auth Sanctum.
- `gomadid/routes/web.php` — halaman Blade per role.

**Cara cepat dapat:** generate tabel `method | path | middleware/role` langsung dari file route (`php artisan route:list --json` → konversi ke Markdown). Ini bisa dikerjakan tanpa akses produksi.

**Yang masih kurang:**
- Belum ada tabel role × endpoint dalam bentuk manusiawi.
- Belum ada penanda endpoint mana yang **sudah dipakai mobile app** vs hanya web — penting untuk menentukan mana yang wajib ada di Core baru sejak hari pertama.
- Webhook masuk (Midtrans) dan endpoint WA service (`gomad-baileys`: `/send`, `/send-bulk`, `/status`, `/health`) perlu didaftarkan terpisah — ini "endpoint" tanpa role user.

---

## P0-6 · Stack & infrastruktur

**Yang dibutuhkan:** framework backend, database, queue, storage, hosting, git repo; dan app mobile itu native/PWA/mobile-web?

**Kenapa:** Menentukan seberapa banyak yang bisa diselamatkan.

**Yang sudah bisa dijawab dari repo:**

| Komponen | Nilai |
|---|---|
| Backend | **Laravel 11** (PHP) — API + Web Blade dalam satu aplikasi |
| Database | **MySQL** (Aiven MySQL, cloud) |
| Queue | Tabel `jobs` ada (migration `2026_07_16_133704`); driver nyata **perlu konfirmasi** (sync vs redis vs database) |
| Storage upload | **Cloudinary** (`CloudinaryService`) |
| Hosting backend | **Render** (Docker) → `web.gomad.id` |
| WhatsApp gateway | **Node.js + Baileys** (`gomad-baileys/`) → Railway, dengan fallback berantai |
| Payment | **Midtrans** — 4 webhook: booking, disbursement/withdrawal, settlement, topup (status: sandbox) |
| Push & email | FCM + Gmail SMTP |
| Mobile app | ⚠️ **Flutter native** (`gomad_app/`) — **bukan PWA**. Hanya untuk Customer & Driver; role lain ditolak saat login |
| iOS | Folder `ios/` ada, **perlu konfirmasi** apakah benar-benar dipakai/di-build |
| Version control | Repo ini (`go-go-gomad`) — monorepo berisi backend, app, gateway WA, landing, demo |

**Yang masih kurang / perlu konfirmasi:**
- ⚠️ **Driver queue sesungguhnya** di produksi (ada `jobs` table, tapi apakah `QUEUE_CONNECTION=database` dipakai?). Ini menentukan apakah Core baru butuh worker/queue infra.
- ⚠️ **Apakah `gomadid-back/` (backup/staging) masih dipakai** atau bisa dibuang.
- **Backup DB & retensi** — siapa yang backup, seberapa sering, di mana.
- **Domain & DNS** mana saja yang aktif (gomad.id, web.gomad.id, demo.gomad.id, dll).
- **Biaya infra bulanan** — berguna untuk keputusan "rebuild vs patching".
- **Kredensial**: cukup dicatat *lokasinya*, jangan nilainya.

---

## Definisi "P0 selesai"

Tahap 1 bisa dimulai kalau keenam item di atas sudah ada — atau sudah ditandai eksplisit **"tidak ada, dan ini keputusannya"**. Yang **tidak** boleh terjadi adalah item P0 dianggap selesai hanya karena "ada di kode tapi belum ditulis".
