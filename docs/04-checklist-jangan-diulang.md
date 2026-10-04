# 04 — Checklist "Jangan Diulang" untuk Rebuild GoMad

> **Konteks:** `go-go-gomad` adalah **referensi saja** — bukan basis kode untuk direplikasi.
> Isinya berantakan, jadi tidak ada yang kita warisi kecuali **pengetahuan domain Mad Trans**.
>
> Karena itu, temuan-temuan ini **bukan daftar perbaikan di kode lama**, melainkan
> **kriteria lulus untuk sistem baru**. Kalau sebuah temuan tidak dicegah lewat desain,
> dia akan lahir kembali di GoMad baru — dengan wajah yang sama.

---

## Cara pakai dokumen ini

Untuk setiap butir, ada **kriteria lulus** yang bisa diuji. Sebuah butir dianggap beres
hanya kalau ada **tes otomatis** yang membuktikannya, bukan kalau "sudah diperhatikan".

---

## A · Uang & Ledger

Ini area paling berbahaya. Semua temuan 🔴 ada di sini.

| # | Jangan diulang | Kriteria lulus |
|---|---|---|
| A1 | **Transaksi tanpa kategori.** Legacy: `wallet_transactions` hanya `credit`/`debit`, pembeda cuma `reference_type` | Setiap mutasi punya: `account_id`, `category`, `reference_type` + `reference_id`, `direction`, `amount`, `balance_after` |
| A2 | **`balance_before`/`after` menunjuk kantong berbeda tanpa penanda akun** → ledger tidak bisa direkonsiliasi | `SUM(mutasi)` per akun **merekonstruksi** saldo akun itu. Ada tes yang membuktikannya |
| A3 | **PPN dicatat sebagai pendapatan** | Akun terpisah: `Revenue: Service Fee`, `Revenue: Platform Fee`, `Tax Payable: PPN Keluaran`. PPN **tidak pernah** masuk Revenue |
| A4 | **Tidak ada PPN sama sekali** | PPN 11% dihitung dari total fee platform, punya akun sendiri, dan disetor terpisah |
| A5 | **Hold disimpan sebagai kolom saldo** | Hold = **record** `wallet_holds` dengan `unique(source_type, source_id, reason)` |
| A6 | **Deposit terkunci permanen** — tidak bisa ditarik selamanya | Ada rumus `withdrawable`; deposit bisa ditarik selama `held` tetap terpenuhi |
| A7 | **Double release** oleh cron (tidak ada guard) | Setiap pelepasan hold idempoten. Tes: panggil 2× → efek 1× |
| A8 | **Idempotency bolong di disbursement/withdrawal** — tanpa signature & cek status → refund berulang | **Semua** callback: verifikasi signature + state guard + unique constraint. Tes: kirim callback `failed` 3× → dana tidak berubah 3× |
| A9 | **Tunggakan disimpan di JSON** (`payment_detail.cod_fee_outstanding`) | Semua piutang jadi entitas ledger, punya state, bisa ditagih & diaudit |
| A10 | **Refund tunai warung tanpa pemroses** — `refund_pending` menggantung | Setiap state refund punya jalur keluar. Tidak ada state yang bisa menggantung tanpa batas |
| A11 | **Dua sumber kebenaran hold COD** (`route.cod_min_deposit` vs `schedule.cod_min_balance`) | Satu sumber kebenaran. Tidak ada aturan bisnis yang hidup di dua tempat |
| A12 | **Settlement tanpa PPN & tanpa potongan warung tercatat** | Settlement menampilkan rincian: fare agency, fee platform, PPN, komisi warung, hold, net |

---

## B · Inventori Kursi

| # | Jangan diulang | Kriteria lulus |
|---|---|---|
| B1 | **Tidak ada kunci kursi & TTL.** Kursi ditahan booking `pending` sampai pembayaran kedaluwarsa | Kunci kursi eksplisit + TTL, dilepas oleh scheduler |
| B2 | **COD tanpa kedaluwarsa** → kursi nyangkut selamanya | COD punya TTL. Tidak ada jalur booking tanpa batas waktu |
| B3 | **Tidak ada cutoff penjualan** → bisa booking 1 menit sebelum berangkat | Cutoff configurable, ditegakkan di satu tempat |
| B4 | **Tidak ada unique constraint kursi** + auto-assign menulis ke salinan `foreach` → kursi dobel | `unique(schedule_id, seat_number)` untuk booking aktif. Tes: pesan kursi yang sama 2× → ditolak |
| B5 | **Kapasitas dihitung dari `sum(total_passengers)`** | Satu sumber: `seat_layout` kendaraan |
| B6 | **Kapasitas per jadwal, bukan per segmen** → penumpang A→B dan C→D saling makan kursi | Model kursi dirancang `kursi × segmen` sejak awal, walau MVP masih per jadwal |
| B7 | **Logika kapasitas tersebar & duplikat** di controller Web, API, dan service | Satu komponen menegakkan kapasitas. Controller tidak boleh punya logika sendiri |

---

## C · Aturan Bisnis Terpusat

| # | Jangan diulang | Kriteria lulus |
|---|---|---|
| C1 | **Kolom hidup, logika mati** — 7 kolom ada di DB tapi tidak pernah jalan | Setiap kolom config wajib punya pemakai, atau **dihapus**. Tidak ada kolom dekoratif |
| C2 | **Config mati** (`schedule_min_days_before = 30` tidak dipakai) | Setiap entri config punya tes atau dihapus |
| C3 | **`credit_limit` boleh saldo minus** — bertentangan dengan deposit gating | Tidak ada saldo negatif. Tidak ada kredit platform di MVP |
| C4 | **Koin COD** (saldo virtual) | Tidak ada mata uang bayangan. Hanya `available` + `held` |
| C5 | **`cost_bearer` promo diabaikan** → agency selalu menanggung diskon platform | `cost_bearer` dihormati. Tes: promo platform → `agency_net` penuh, `platform_revenue` berkurang |
| C6 | **Promo referral platform memotong agency** (bug komersial) | Sama seperti C5, dibuktikan dengan tes |
| C7 | **Permission hanya per role tunggal**, tanpa tabel permission | Aksi (bukan hanya role) terdaftar eksplisit. Siap untuk agency multi-user & finance terpisah |

---

## D · State & Konsistensi

| # | Jangan diulang | Kriteria lulus |
|---|---|---|
| D1 | **Tidak ada `ScheduleStatus`/`TripStatus`** — status tersebar di 3 kolom + timestamp | Setiap agregat punya state machine eksplisit (enum + transisi legal) |
| D2 | **Web vs API menghasilkan status akhir booking berbeda** (`confirmCod`) | Satu state machine dipakai **semua** kanal. Tes: kanal berbeda → status akhir sama |
| D3 | **Jadwal tidak dinonaktifkan saat selesai** → memicu double release | Transisi state mengurus pelepasan hold secara idempoten |

---

## E · Data & Operasi

| # | Jangan diulang | Kriteria lulus |
|---|---|---|
| E1 | **Migration drift** — skema produksi ≠ migration (ada `drop_max_overload`, 4 migration index, perubahan tipe) | Skema = sumber kebenaran, diverifikasi otomatis di CI |
| E2 | **Tidak ada observabilitas transaksi uang** | Setiap mutasi uang punya jejak: siapa, kapan, dari mana, untuk apa |
| E3 | **Error 422 memakai key `data`, klien baca `errors`** → pesan asli tertelan | Kontrak error API tunggal, dipakai semua klien |
| E4 | **Midtrans sandbox** — uang belum pernah benar-benar berpindah | Ada rencana transaksi nyata pertama + prosedur rekonsiliasi tertulis |

---

## F · Yang harus DIUJI, bukan hanya dirancang

Lima skenario ini wajib punya tes otomatis sebelum MVP dianggap selesai:

1. **Callback duplikat** — kirim callback pembayaran/disbursement yang sama 3× → saldo berubah **sekali**
2. **Race condition kursi** — 2 permintaan kursi yang sama secara bersamaan → **satu** berhasil
3. **Saldo kurang** — agency mencoba COD tanpa deposit cukup → **ditolak di satu titik**, tanpa memblokir fitur lain
4. **Rekonsiliasi ledger** — jumlah mutasi merekonstruksi saldo setiap akun, untuk semua skenario
5. **Rincian settlement** — fare agency + fee platform + PPN + komisi warung + hold = net, tanpa selisih 1 rupiah

---

## Ringkasan prioritas

| Prioritas | Butir | Alasan |
|---|---|---|
| 🔴 **Wajib di MVP** | A1–A9, A11, B1, B2, B4, B5, B7, C1, C3, C4, D2, F1–F5 | Menyangkut uang langsung atau menyebabkan kebocoran yang sulit dideteksi |
| 🟠 **MVP, boleh bertahap** | A10, A12, B3, B6, C5–C7, D1, D3, E1, E3 | Penting, tapi tidak langsung merusak uang |
| 🟡 **Setelah MVP** | E2, E4 | Butuh data atau transaksi nyata |
