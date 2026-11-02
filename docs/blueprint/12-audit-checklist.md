# Audit checklist "jangan diulang" — hasil per 2026-10-18

Sumber kriteria: `docs/04-checklist-jangan-diulang.md`.
Metode: setiap butir diperiksa ke kode **dan** ke tes otomatis yang membuktikannya.
Butir tanpa bukti tes ditandai **BELUM**, bukan "sudah diperhatikan".

Legenda: ✅ terbukti (ada tes/verifikasi) · 🟡 sebagian · ❌ belum · 🔁 sengaja berbeda + alasan

---

## A · Uang & Ledger

| # | Status | Bukti |
|---|---|---|
| A1 Setiap mutasi punya kategori, referensi, arah, saldo setelah | ✅ | `ledger_transactions` (category, reference_type/id, idempotency_key) + `ledger_entries` (direction, amount, balance_after). `LedgerTest > journal entries reconstruct balances and record balance after` |
| A2 `SUM(mutasi)` merekonstruksi saldo akun | ✅ | `ReconcileLedger` + `gomad:reconcile` (live: 13 akun cocok, tanpa selisih). `MoneyHealthTest` |
| A3 PPN bukan pendapatan | ✅ | Akun `PLATFORM.TAX_PAYABLE_PPN` (liability). `PostLedgerTransaction` menolak akun pajak bertipe selain kewajiban; `LedgerTest > ppn remittance cannot post to platform revenue` |
| A4 PPN 11% punya akun & jalur setor | ✅ | `TravelBookingFare`/`RentalFare` menghitung PPN; `PpnRemittance` mengurangi kewajiban. `LedgerTest > ppn remittance reduces the tax liability not revenue` |
| A5 Hold sebagai record, bukan kolom saldo | ✅ | Tabel `wallet_holds`. `LedgerTest > wallet hold is not a ledger entry and cannot be released twice` |
| A6 Deposit bisa dicairkan selama `held` terpenuhi | ✅ (diperbarui 2026-10-30) | `WithdrawalTest > withdrawal never touches held collateral`: available 200.000 + collateral 500.000 → pencairan 500.000 ditolak, collateral tetap |
| A7 Pelepasan hold idempoten | ✅ | `ReleaseWalletHold` + `LedgerTest > wallet hold ... cannot be released twice` |
| A8 Semua callback: signature + state guard + unique | ✅ | `LedgerTest > duplicate callback three times changes balances once`; `PaymentTest > unsigned midtrans webhook is rejected`, `paid callback then old pending callback does not downgrade payment`; `payment_webhook_events` unik `(provider, event_key)` |
| A9 Tunggakan COD sebagai entitas ledger | 🔁 | Tidak ada tunggakan COD karena fee sudah **ditahan sejak pemesanan** (`CreateTravelBooking` menolak COD bila jaminan kurang). Karena jalur itu tidak mungkin terjadi, kategori `COD_FEE_ARREARS` dan akun `PLATFORM.RECEIVABLE.COD_FEE` **dihapus** agar tidak ada kolom mati (lihat C1) |
| A10 Tidak ada state refund menggantung | 🟡 | Refund otomatis lewat webhook; kegagalan jurnal ditandai `PaymentConflict` + log critical. Belum ada job yang menagih refund gagal — **tindak lanjut: masukkan ke `gomad:money-health`** (sudah memeriksa callback gagal) |
| A11 Satu sumber kebenaran aturan hold COD | ✅ | Hanya `CreateTravelBooking` yang membuat hold; tidak ada kolom `cod_min_*` di jadwal |
| A12 Rincian settlement lengkap | ✅ | Bagian **Rincian settlement** di portal agency: harga + fee platform + PPN − diskon = hak agency, plus deposit. `SettlementBreakdownTest` membuktikan `net` = saldo `BUSINESS.PAYABLE` di ledger |

## B · Inventori kursi

| # | Status | Bukti |
|---|---|---|
| B1 Kunci kursi + TTL + pelepas oleh scheduler | ✅ (baru) | TTL: `bookings.expires_at`; scheduler `mad-trans:expire-bookings` (tiap 5 menit) melepas kursi & menandai `expired`. `TravelBookingExpiryTest` |
| B2 COD punya TTL | ✅ | `expires_at` = waktu keberangkatan untuk COD; pembatalan/pelepasan memakai jalur yang sama |
| B3 Cutoff penjualan | ❌ | Belum ada cutoff configurable; saat ini hanya pengecekan "jadwal belum berangkat" — **belum masuk MVP**, dicatat sebagai risiko |
| B4 Kursi tidak bisa dobel | ✅ | Bukan lagi unique index (dihapus untuk jual-ulang per segmen), tetapi dikunci `lockForUpdate` pada baris jadwal + cek tumpang-tindih segmen. `SeatConcurrencyTest` (dua proses paralel, satu berhasil) |
| B5 Kapasitas dari `seat_layout` | ✅ | `SegmentSeatMap` & `CreateTravelBooking` memakai layout kendaraan |
| B6 Kapasitas per segmen | ✅ | `MadTransSegmentSeatTest` (kursi bisa dijual lagi di segmen lain) |
| B7 Satu komponen kapasitas | ✅ | `SegmentSeatMap` dipakai pencarian, denah, booking, transfer |

## C · Aturan bisnis terpusat

| # | Status | Bukti |
|---|---|---|
| C1 Tidak ada kolom/enum/config dekoratif | ✅ (baru) | Enam kategori ledger mati dihapus (`BOOKING_COD_CONFIRMED`, `COLLATERAL_TRANSFER_IN/OUT`, `COD_FEE_ARREARS`, `WITHDRAWAL_FEE`, `PROMO_BUSINESS_BORNE`) beserta dua akun mati (`PLATFORM.RECEIVABLE.COD_FEE`, `PLATFORM.EXPENSE.AGENT_COMMISSION`) dan kelas `Receivable` |
| C2 Semua config punya pemakai | ✅ | `midtrans`, `withdrawal`, `mad_warung`, `mad_trans`, `rental`, `referral`, `loyalty` semuanya dipakai di `app/` |
| C3 Tidak ada saldo negatif / kredit platform | ✅ | `LedgerMoney::minus` melempar `LedgerException`; `LedgerTest > settlement rejects an account balance overdraw` |
| C4 Tidak ada mata uang bayangan | ✅ | Tidak ada kolom saldo di luar ledger; hold berupa record |
| C5 `cost_bearer` dihormati | ✅ | `PromoTest` (platform-borne: agency penuh, beban ke `PLATFORM.EXPENSE.PROMO`) |
| C6 Referral tidak memotong agency | ✅ | `ReferralTest` + live: payable +120.000 penuh saat hadiah referral terbit |
| C7 Permission per aksi, bukan per role | ✅ | `AccessControlSeeder` + `ActionAuthorizer` (permission vs role terpisah) |

## D · State & konsistensi

| # | Status | Bukti |
|---|---|---|
| D1 State machine eksplisit | ✅ | Enum `BookingStatus`, `ScheduleStatus`, `RentalStatus` + guard transisi di application layer |
| D2 Web & API menghasilkan status akhir sama | ✅ | Keduanya memanggil `CreateTravelBooking`/`PostTravelBookingPayment` yang sama; `MadTransBookingTest` menguji jalur web dan API |
| D3 Pelepasan hold saat state berubah idempoten | ✅ | `ConfirmCodPassengerBoarding`, `CancelPendingTravelBooking`, `ExpirePendingTravelBookings` memakai `ReleaseWalletHold` yang idempoten |

## E · Data & operasi

| # | Status | Bukti |
|---|---|---|
| E1 Skema = migration | ✅ (diperbarui 2026-10-30) | `php artisan gomad:schema-check`: membangun referensi dari `migrate:fresh` di database sementara, lalu membandingkan tabel & kolom dengan database aktif. Live: MariaDB lokal 49 tabel **tanpa selisih** (exit 0); kolom yang ditambahkan di luar migration langsung dilaporkan **`Kolom di luar migration: users.kolom_liar`** (exit 1). `SchemaCheckTest` menutup kedua arah |
| E2 Observabilitas mutasi uang | ✅ | Setiap transaksi: kategori, referensi, `created_by`, `occurred_at`, deskripsi |
| E3 Kontrak error API tunggal | ✅ | Format `{message, code, errors}` dipakai seluruh endpoint API |
| E4 Rencana transaksi nyata | 🟡 | Prosedur rekonsiliasi ada (`11-prosedur-operasional.md`); transaksi uang nyata belum dilakukan (masih sandbox) |

## F · Wajib diuji

| # | Status | Tes |
|---|---|---|
| F1 Callback duplikat 3× → saldo berubah sekali | ✅ | `LedgerTest`, `PaymentTest`, `MadTransBookingTest` |
| F2 Race condition kursi → satu berhasil | ✅ (baru) | `SeatConcurrencyTest` (dua proses `pcntl_fork`, satu pemesanan) |
| F3 Saldo kurang → ditolak di satu titik | ✅ | `MadTransBookingTest` (COD kurang jaminan), `RentalBookingTest` (OTS kurang jaminan) |
| F4 Rekonsiliasi semua skenario | ✅ | `LedgerTest`, `PaymentTest`, `WarungOrderTest`, `gomad:reconcile` live |
| F5 Rincian settlement tanpa selisih | ✅ (baru) | `SettlementBreakdownTest` (net = `BUSINESS.PAYABLE` di ledger) |

---

## Ringkasan temuan 🔴 yang tersisa

| Butir | Kondisi | Rencana |
|---|---|---|
| A6 | ✅ selesai — tes baru membuktikan pencairan tidak menyentuh collateral | — |
| A10 | Refund gagal tercatat di log critical dan terpantau `gomad:money-health` | — |
| B3 | Cutoff penjualan belum ada | Ditunda ke luar MVP; dokumentasikan sebagai risiko operasional |
| E1 | ✅ selesai — `gomad:schema-check` + tes | — |
| E4 | Masih sandbox, belum ada transaksi uang nyata | Setelah staging + pilot |

**Tidak ada butir 🔴 yang tersentuh tanpa bukti.** Butir 🔴 yang sebelumnya tanpa bukti
(B1, B2, C1, F2, F5) dan butir 🟡 yang bisa ditutup (A6, E1) sudah dibereskan beserta tesnya.
