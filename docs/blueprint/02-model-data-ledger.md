# Blueprint 02 — Model Data & Ledger

> Skema konkret. Rujukan desain: `docs/05-desain-gomad-core.md` · pola peran: `docs/09`.
> **Aturan uang:** tidak ada tabel saldo liar · tidak ada entri ledger yang diubah/dihapus.

---

## 1. Prinsip pemodelan

| # | Prinsip | Konsekuensi |
|---|---|---|
| 1 | **Warung, Grosir, Agency semuanya `business`** | Satu tabel, banyak tipe — bukan tiga tabel mirip |
| 2 | **Peran ada di keanggotaan, bukan di user** | Satu orang bisa punya banyak peran di banyak usaha |
| 3 | **Keanggotaan M:N sejak awal** | Menambah multi-agency nanti tanpa migrasi |
| 4 | **Uang selalu `decimal(18,2)`** | Tidak pernah `float` |
| 5 | **Entri ledger bersifat abadi** | Koreksi lewat transaksi pembalik, bukan `update`/`delete` |
| 6 | **Setiap query dibatasi ke usaha pengguna** | Kebocoran data lintas usaha = risiko utama |

---

## 2. Konvensi

| Aspek | Konvensi |
|---|---|
| Nama tabel | Jamak, bahasa Inggris — konsisten dengan Laravel |
| Kunci utama | `id` bigint *auto-increment* (internal) |
| **Kode manusiawi** | Kolom terpisah ber-indeks unik, **dipisah per vertical** — lihat skema di bawah |
| Kunci asing | `xxx_id`, selalu ber-indeks |
| Status | Kolom `status` bertipe string + **enum PHP**, bukan `enum` MySQL |
| Waktu | UTC di database, ditampilkan sesuai zona pengguna |
| Soft delete | Hanya entitas yang perlu diaudit. **Tidak** untuk entri ledger |
| Migrasi | Dipisah: `core/` · `mad-warung/` · `mad-trans/` |

**Kenapa status pakai string + enum PHP, bukan `enum` MySQL:** menambah nilai enum MySQL butuh migrasi; enum PHP cukup ubah kode. Sistem lama memakai `enum` MySQL dan itu menyulitkan.

### Skema kode referensi (dipisah per vertical)

```
{VERTIKAL}-{JENIS}-{YYMM}-{URUT 4 digit}
```

| Contoh | Arti |
|---|---|
| `MW-OR-2610-0001` | Mad Warung · pesanan (order) |
| `MW-PY-2610-0001` | Mad Warung · pembayaran |
| `MT-TR-2610-0001` | Mad Trans · booking travel |
| `MT-RN-2610-0001` | Mad Trans · rental |
| `MT-TF-2610-0001` | Mad Trans · transfer penumpang |
| `WD-2610-0001` | Core · pencairan (withdrawal) |
| `ST-2610-0001` | Core · settlement |
| `RF-2610-0001` | Core · refund |

**Alasan dipisah:** saat membaca log, tiket dukungan, atau mutasi, **vertical dan jenis transaksi langsung terbaca** tanpa perlu membuka baris database. Dan menambah vertical baru cukup memakai prefix baru — tidak mengganggu yang lama.

---

## 3. Inti: Identitas & Keanggotaan

```
users
  id · name · phone (unik) · email (unik, nullable) · password
  status (active|suspended) · phone_verified_at · last_login_at
  ── phone adalah identitas utama (konteks Indonesia), email opsional

businesses
  id · type (warung|grosir|agency) · name · code (unik)
  owner_user_id (nullable)
  alamat: province_id · city_id · district_id · village_id · address · latitude · longitude
  status (draft|active|suspended) · verified_at

business_members
  id · business_id · user_id
  role (pemilik|penjaga|driver|staf)
  status (aktif|nonaktif) · joined_at · left_at
  UNIQUE(business_id, user_id, role)

permissions
  id · code (aksi, mis. "order.create") · label · group

role_permissions
  role · permission_id
```

### Pemetaan peran nyata → model

| Peran nyata | Diwakili oleh |
|---|---|
| Pemilik warung | keanggotaan `pemilik` pada bisnis `warung` — **bisa banyak** |
| Penjaga warung | keanggotaan `penjaga` pada bisnis `warung` — **bisa banyak** |
| Grosir | `business` tipe `grosir` |
| Agency travel | `business` tipe `agency` |
| Driver | keanggotaan `driver` pada bisnis `agency` |
| Penumpang / penyewa | `user` **tanpa** keanggotaan |
| Admin / Ops | peran platform (di luar keanggotaan bisnis) |

### ⚠️ Aturan multi-tenant — paling penting di dokumen ini

Setiap query ke tabel transaksional **wajib** dibatasi ke usaha yang boleh diakses pengguna.

**Ditegakkan lewat *global scope* + policy terpusat, bukan disiplin manual.** Kalau diserahkan pada disiplin, cepat atau lambat akan ada satu query yang lupa dan membocorkan data usaha lain.

---

## 4. Ledger

```
ledger_accounts
  id · code (unik) · owner_type (platform|business|user) · owner_id (nullable)
  account_class (asset|liability|revenue|expense|equity|receivable)
  purpose (available|collateral|cash|revenue|…)
  balance_cached decimal(18,2) · currency (IDR) · is_active
  UNIQUE(owner_type, owner_id, purpose)

ledger_transactions
  id · category · reference_type · reference_id
  idempotency_key (UNIK)          ← kunci anti-duplikat
  amount decimal(18,2) · status (posted|reversed)
  description · occurred_at · created_by

ledger_entries
  id · transaction_id · account_id
  direction (debit|credit) · amount · balance_after
  UNIQUE(transaction_id, account_id, direction)

wallet_holds                                    ← pengganti kolom saldo-hold
  id · account_id · source_type · source_id · reason
  amount · status (active|released|consumed|expired)
  expires_at · released_at · transaction_id
  UNIQUE(source_type, source_id, reason)        ← mencegah pelepasan ganda
```

### Bagan akun

| Kode | Kelas | Fungsi |
|---|---|---|
| `PLATFORM.CASH` | asset | Kliring gateway / rekening bank |
| `PLATFORM.PAYABLE_TO_BUSINESS` | liability | Hak mitra yang belum disetor |
| `PLATFORM.REVENUE.SERVICE_FEE` | revenue | Biaya layanan |
| `PLATFORM.REVENUE.PLATFORM_FEE` | revenue | Biaya platform |
| `PLATFORM.REVENUE.WITHDRAWAL_FEE` | revenue | Biaya admin pencairan |
| `PLATFORM.TAX_PAYABLE_PPN` | liability | **PPN — bukan pendapatan** |
| `PLATFORM.RECEIVABLE_COD` | receivable | Piutang fee dari transaksi COD |
| `BUSINESS.AVAILABLE:{id}` | liability | Saldo bebas mitra (agency & grosir) |
| `BUSINESS.COLLATERAL:{id}` | liability | Saldo mengendap (deposit) |

**Tanpa akun:** warung & penumpang. Keduanya membayar per transaksi lewat gateway, tidak menyimpan saldo.

### Kategori transaksi

18 kategori — daftar lengkap di `docs/05` bagian 4. Yang **wajib** ada di Rilis 1:

`BOOKING_PAID_ONLINE` · `BOOKING_COD_CONFIRMED` · `RENTAL_PAID` · `COLLATERAL_TOPUP` · `COLLATERAL_TRANSFER_IN` · `COLLATERAL_TRANSFER_OUT` · `COD_FEE_COLLECTED` · `COD_FEE_ARREARS` · `SETTLEMENT_TO_BUSINESS` · `WITHDRAWAL_REQUEST` · `WITHDRAWAL_FEE` · `WITHDRAWAL_REVERSAL` · `REFUND_APPROVED` · `PPN_REMITTANCE` · `ORDER_PAID_TO_GROSIR` · `PROMO_PLATFORM_BORNE` · `PROMO_BUSINESS_BORNE` · `ADJUSTMENT`

### Aturan ledger yang tidak bisa ditawar

| # | Aturan |
|---|---|
| L1 | Setiap transaksi minimal **2 entri** yang saling menutup |
| L2 | `SUM(debit) = SUM(credit)` per transaksi — diperiksa tes |
| L3 | `balance_after` dicatat **per akun**, bukan per kantong ambigu |
| L4 | Entri **tidak pernah** diubah atau dihapus |
| L5 | Idempotency diperiksa **di dalam** transaksi database, dengan `lockForUpdate` juga **di dalam** |
| L6 | `wallet_holds` **tidak** menghasilkan entri — hold bukan perpindahan uang |
| L7 | PPN **tidak pernah** masuk akun pendapatan |

---

## 5. Mad Trans — tabel

```
routes          id · business_id · origin_city · destination_city ·
                is_system_generated · status
route_stops     id · route_id · city_id · stop_order · name
route_pricing   id · route_id · origin_stop_id · destination_stop_id · price
                (harga per SEGMEN — bukan satu harga per rute)

vehicles        id · business_id · name · plate_number · seat_layout (JSON) ·
                capacity · status
schedules       id · business_id · vehicle_id · route_id · driver_id (nullable)
                departure_date · departure_time · travel_class
                price_per_seat · status · proposed_by · approved_by
                allow_cod · payment_methods · started_at · finished_at
schedule_stops  id · schedule_id · stop_order · city_id · pickup_zone_id

bookings        id · code · schedule_id · customer_id
                origin_stop_id · destination_stop_id · route_pricing_id
                pickup_address + lat/lng · destination_address + lat/lng
                total_passengers · base_price · service_fee · platform_fee
                discount_amount · ppn_amount · total_price · status
                expires_at                          ← pengganti TTL kursi (lihat catatan)
booking_passengers  id · booking_id · name · seat_number · phone
                    UNIQUE(schedule_id, seat_number) untuk booking aktif
```

**Catatan penting — kunci kursi:** keputusan di `docs/05` menyatakan penahanan kursi = durasi pembayaran. Karena itu **tidak ada tabel kunci kursi terpisah**; yang menahan kursi adalah `bookings.expires_at`. Scheduler melepas kursi yang kedaluwarsa. Konsekuensinya: `booking_passengers` **wajib** punya kunci unik per jadwal — inilah yang mencegah kursi dobel.

```
rentals         id · code · business_id · vehicle_id · customer_id
                rental_type (self_drive|with_driver) · start_at · end_at
                price · service_fee · platform_fee · ppn_amount · total_price
                deposit_amount · deposit_status · status
rental_documents    id · rental_id · type (ktp|sim|selfie) · file · verified_at
passenger_transfers id · booking_id · from_schedule_id · to_schedule_id
                    fee · status · approved_by
driver_assignments  id · schedule_id · driver_id (keanggotaan) · status
```

---

## 6. Mad Warung — tabel

```
grosir_catalogs     id · business_id (grosir) · nama katalog · status ·
                    dikelola_oleh (gomad_awal_grosir_update)   ← hibrida, diputuskan
products            id · category_id · name · unit · barcode (nullable)
product_prices      id · product_id · business_id (grosir) · price · min_order ·
                    stok_tersedia · updated_at
                    (harga BERBEDA per grosir — bukan satu harga global)

member_limits       id · business_id · user_id (penjaga) ·
                    daily_amount_limit · per_order_limit · requires_approval_above
                    UNIQUE(business_id, user_id)             ← plafon per penjaga

shifts              id · business_id · user_id (penjaga) · start_at · end_at ·
                    status (terjadwal|berjalan|selesai) · catatan

orders              id · code · business_id (warung) · grosir_id ·
                    created_by (penjaga) · approved_by (pemilik, nullable)
                    total_amount · status · approved_at · paid_at
                    delivery_method (antar_grosir|ambil_sendiri) ·
                    received_confirmed_by · received_at
order_items         id · order_id · product_id · qty · unit_price · subtotal
order_approvals     id · order_id · requested_by · decided_by · decision ·
                    reason · decided_at
```

### Titik kendali yang tidak boleh hilang

| Kendali | Di mana |
|---|---|
| Plafon penjaga | `member_limits` — diperiksa saat pesanan dibuat |
| Approval pemilik | `order_approvals` — bila melewati `requires_approval_above` |
| Jejak pelaku | `orders.created_by` + `shifts` → siapa bertugas saat itu |
| Bukti penerimaan | `orders.received_confirmed_by` + `received_at` — pengaman karena GoMad membayar grosir lebih dulu |

---

## 7. State machine

Setiap agregat punya **satu** state machine, didefinisikan sebagai enum PHP + transisi legal. Dipakai **semua kanal** — tidak boleh web dan API menghasilkan keadaan akhir berbeda.

```
ScheduleStatus   draft → pending_approval → published → in_progress → completed
                 pending_approval → rejected        (oleh pemilik/agency)
                 draft|pending_approval|published → cancelled

OrderStatus      draft → pending_approval → approved → pending_payment
                 → paid → preparing → ready
                 → in_delivery | ready_for_pickup
                 → received → completed
                 (+ rejected, cancelled)

BookingStatus    pending → confirmed → paid → on_going → completed
                 (+ cancelled, expired)

RentalStatus     pending → paid → active → returned → completed (+ cancelled)

HoldStatus       active → released | consumed | expired

SettlementStatus pending → paid → verified (+ overdue)

WithdrawalStatus pending → approved → processing → completed
                 (+ rejected, failed → reversal)
```

**Perbedaan dari sistem lama:** `ScheduleStatus` dan `OrderStatus` kini **eksplisit**. Di sistem lama status jadwal tersebar di tiga kolom + timestamp, dan itu sumber bug.

---

## 8. Aturan lintas tabel

| # | Aturan | Alasan |
|---|---|---|
| T1 | Tabel transaksional punya `business_id` yang relevan | Dasar pembatasan multi-tenant |
| T2 | Semua FK ber-indeks | Performa |
| T3 | Indeks pada kolom `status` yang sering difilter | Daftar pekerjaan harian |
| T4 | `idempotency_key` unik di `ledger_transactions` | Anti-duplikat |
| T5 | `UNIQUE(schedule_id, seat_number)` untuk booking aktif | Anti-kursi-dobel |
| T6 | `UNIQUE(source_type, source_id, reason)` di `wallet_holds` | Anti-pelepasan ganda |
| T7 | Tidak ada kolom yang "dipersiapkan nanti" | Pelajaran dari sistem lama: 7 kolom mati |
| T8 | Setiap kolom config wajib punya pemakai, atau tidak ditulis | Pelajaran: `schedule_min_days_before` tidak pernah dipakai |

---

## 9. Status keputusan

### Sudah diputuskan (2026-10-04)

| # | Item | Keputusan |
|---|---|---|
| 1 | **Pemelihara katalog grosir** | ✅ **Hibrida** — GoMad isi katalog awal, grosir update harga & stok |
| 2 | **Asumsi PPN** | ✅ **Ditambahkan ke pembeli** → `bookings.ppn_amount` & `orders.ppn_amount` dibebankan ke pembeli |
| 3 | **Fallback pembayaran** | ✅ Metode default + QRIS/VA → tidak mengubah skema, hanya alur |

### Masih menunggu

| # | Menunggu | Memengaruhi |
|---|---|---|
| 1 | **Wilayah pertama** dari 3 wilayah sasaran | Bentuk data referensi wilayah & migrasi |
| 2 | **Service fee Mad Warung** angkanya | Tabel parameter |
| 3 | **Plafon default penjaga** | Nilai awal `member_limits` |
| 4 | **Prefix kode referensi** per vertical | Format `orders.code` & `bookings.code` |
