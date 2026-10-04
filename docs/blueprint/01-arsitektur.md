# Blueprint 01 — Arsitektur & Batas Modul

> Dasar untuk semua dokumen blueprint lain.
> Konsisten dengan: `docs/09` (pola peran) · `docs/05` (Core) · `docs/00` (aturan kerja)

---

## 1. Prinsip

| # | Prinsip | Konsekuensi |
|---|---|---|
| 1 | **Core stabil, vertical bisa berubah** | Vertical bergantung pada Core; **tidak pernah** sebaliknya |
| 2 | **Satu logika, banyak kanal** | Web dan API memanggil Action yang sama. Tidak ada logika di controller |
| 3 | **Domain tidak tahu web** | Lapisan domain tidak menyentuh session, cookie, Inertia, atau HTTP |
| 4 | **Batas ditegakkan mesin**, bukan kesepakatan | Pelanggaran batas **menggagalkan CI** |
| 5 | **Kesiapan mobile dijaga sejak awal** | Setiap fitur punya endpoint API, meski web memakai Inertia |

---

## 2. Struktur workspace

```
~/gomad/
├── api/                     ← repo 1 · Laravel + Inertia + Vue
├── mobile/                  ← repo 2 · Flutter (fase 9)
└── docs/                    ← sumber kebenaran & penghubung netral
```

**Dua repo terpisah** → `docs/blueprint/03-kontrak-api.md` adalah **satu-satunya antarmuka** antara keduanya.

---

## 3. Struktur `api/`

```
api/
├── app/
│   ├── Core/                          ← fondasi bersama
│   │   ├── Identity/                  ← pengguna, usaha, keanggotaan
│   │   ├── Access/                    ← peran, izin, kebijakan
│   │   ├── Ledger/                    ← akun, transaksi, entri, hold
│   │   ├── Settlement/                ← siklus & aturan pembagian
│   │   ├── Payment/                   ← gateway, webhook, pencairan
│   │   ├── Notification/              ← WhatsApp, in-app
│   │   └── Support/                   ← base class, value object, helper
│   │
│   ├── Modules/                       ← vertical
│   │   ├── MadWarung/
│   │   └── MadTrans/
│   │
│   ├── Http/
│   │   ├── Web/                       ← Inertia + Vue, per peran
│   │   └── Api/V1/                    ← endpoint untuk mobile & integrasi
│   └── Providers/
│
├── routes/
│   ├── web/          pemilik.php · penjaga.php · grosir.php · customer.php · driver.php · admin.php
│   └── api/v1/       core.php · mad-warung.php · mad-trans.php
│
├── resources/js/     Pages/ (Inertia) · Components/ · Layouts/
├── database/migrations/   core/ · mad-warung/ · mad-trans/
└── tests/            Core/ · Modules/ · Arch/
```

### Isi setiap modul & Core

```
<NamaDomain>/
├── Domain/            entitas · value object · domain event   ← PHP murni, tanpa Eloquent
├── Application/       Actions (use case) · DTO · Query         ← satu Action = satu use case
├── Infrastructure/    model Eloquent · repository · klien luar
└── Http/              controller · request · resource          ← tipis, ≤ 20 baris
```

---

## 4. Aturan dependensi (ditegakkan CI)

| # | Aturan | Contoh pelanggaran |
|---|---|---|
| **A1** | Vertical **boleh** memakai Core | ✅ `MadWarung` memakai `Core/Ledger` |
| **A2** | Core **tidak boleh** memakai vertical | ❌ `Core/Ledger` meng-import `MadTrans` |
| **A3** | Vertical **tidak boleh** memakai vertical lain | ❌ `MadWarung` meng-import `MadTrans` |
| **A4** | Akses lintas domain **hanya lewat kontrak** (interface) atau **domain event** | ❌ model Eloquent modul lain dipanggil langsung |
| **A5** | **Semua perpindahan uang** lewat `Core/Ledger` | ❌ modul punya tabel `saldo` sendiri |
| **A6** | Controller hanya memanggil **Action** | ❌ logika bisnis di controller |
| **A7** | **Setiap Action** punya endpoint API + tes | ❌ fitur hanya ada di jalur web |

**Cara menegakkan:** uji arsitektur (Pest `arch()`) + analisis statis di CI. Melanggar = CI merah, bukan sekadar catatan di dokumen.

---

## 5. Pola peran

Ditemukan di `docs/09` bahwa pola yang sama berulang di setiap vertical:

```
PEMILIK USAHA   → punya usaha · multi-unit · pegang uang · melihat semua aktivitas
   STAFF        → akun sendiri · punya jadwal · dibatasi kewenangannya
   PIHAK LUAR   → pelanggan atau pemasok
```

### Pemodelan: satu tabel usaha, peran ada di keanggotaan

**Kuncinya: Warung, Grosir, dan Agency semuanya adalah `business` dengan tipe berbeda.**

| Peran nyata | Diwakili oleh |
|---|---|
| Pemilik warung | user dengan keanggotaan `pemilik` pada bisnis bertipe `warung` |
| Penjaga warung | user dengan keanggotaan `penjaga` pada bisnis `warung` |
| Grosir | `business` bertipe `grosir` (+ stafnya sebagai keanggotaan) |
| Agency travel | `business` bertipe `agency` (+ driver sebagai keanggotaan) |
| Driver | user dengan keanggotaan `driver` pada bisnis `agency` |
| Penumpang / penyewa | user **tanpa** keanggotaan — pihak luar |
| Admin / Ops | user dengan peran platform |

**Keuntungan:** "Pemilik warung ≈ pemilik travel" yang kamu sebut di sesi konsep bukan lagi kemiripan konseptual — dia **satu model data**.

### Keanggotaan dirancang M:N sejak awal

```
business_members
  unique(business_id, user_id, role)
```

Satu penjaga bisa menjaga beberapa warung · satu driver bisa terikat beberapa agency nanti — **tanpa migrasi skema**.

---

## 6. Autentikasi & otorisasi

### Autentikasi — dua jalur, satu identitas

| Kanal | Mekanisme |
|---|---|
| Web (Inertia) | Session |
| API (mobile & integrasi) | Token (Sanctum) |

Keduanya menghasilkan **user yang sama**, sehingga Action tidak perlu tahu kanalnya.

### Otorisasi — berbasis aksi, terpusat

- **Izin didaftarkan sebagai aksi** (mis. `order.create`, `withdrawal.approve`), bukan sekadar cek nama role
- Kebijakan terpusat di `Core/Access`, di satu tempat
- **Dilarang** pengecekan role yang tersebar di controller atau view
- Siap untuk peran baru (`finance`, staf agency multi-user) tanpa membongkar apa pun

---

## 7. ⭐ Aturan kesiapan mobile

Ini yang membuat "web dulu" tidak mengunci mobile nanti. **Dijaga sejak Fase 1.**

| # | Aturan | Alasan |
|---|---|---|
| **M1** | Lapisan Domain & Application **tidak boleh** menyentuh session, cookie, Inertia, Blade, atau `Request` HTTP | Kalau tersentuh, logika tidak bisa dipakai mobile |
| **M2** | Setiap Action menerima **DTO eksplisit**, bukan objek Request | Mobile tidak punya Request web |
| **M3** | **Setiap fitur punya endpoint API**, meski web memakai Inertia | Inertia tidak butuh API; mobile butuh. Kalau tidak dibuat sekarang, harus retrofit |
| **M4** | Bentuk respons API **konsisten** untuk semua endpoint | Klien mobile digenerate, jadi bentuknya harus seragam |
| **M5** | Unggah berkas lewat **endpoint**, bukan form web | Mobile tidak punya form |
| **M6** | Tidak ada keadaan yang disimpan di session (kecuali pesan sementara) | Session tidak ada di mobile |
| **M7** | Notifikasi disiapkan **push-ready** sejak awal | Menambah push belakangan berarti mengubah model notifikasi |

**Uji arsitektur khusus M1 & M2** masuk CI: kalau ada `Request`, `session()`, `auth()->user()` di dalam `Domain/` atau `Application/`, CI merah.

---

## 8. Kanal & realtime

| Kebutuhan | Teknologi | Fase |
|---|---|---|
| Halaman web | Inertia + Vue | 0 |
| API mobile | REST `/api/v1` | 1 |
| Antrean latar | Redis + queue worker | 2 |
| Realtime (status pesanan, lokasi) | Reverb (WebSocket) | 3+ |
| Penjadwalan (settlement, pelepasan hold) | Scheduler | 2 |

**Semua endpoint API diberi versi** (`/api/v1`) sejak awal. Web tidak memakai versi, tapi API mobile memerlukannya.

---

## 9. Konvensi kode

| Aspek | Konvensi |
|---|---|
| Nama modul | Pakem mindmap: `MadWarung`, `MadTrans`, `Core/Ledger` — **bukan** istilah baru |
| Satu use case | Satu Action class, satu nama kerja (`CreateOrder`, `ApproveWithdrawal`) |
| Uang | `decimal(18,2)` · Rupiah · **tidak pernah** `float` |
| Kode manusiawi | Prefix seperti `GM-` (pesanan) — untuk referensi manusia, bukan kunci utama |
| Migrasi | Dipisah per domain: `core/`, `mad-warung/`, `mad-trans/` |
| Penamaan tabel | Jamak, Inggris (`businesses`, `orders`) — konsisten dengan Laravel |
| Soft delete | Hanya untuk entitas yang perlu diaudit; **jangan** untuk entri ledger |

---

## 10. Yang harus sudah ada sebelum Fase 1 dimulai

- [ ] Dua repo dibuat, `api/` jalan di staging ber-URL
- [ ] CI menjalankan: lint · tes · **uji arsitektur** · migrasi
- [ ] Uji arsitektur untuk aturan A1–A7 dan M1–M2 sudah menuliskan batasnya (boleh awalnya kosong, tapi kerangkanya ada)
- [ ] Kontrak API punya tempat resmi + cara generate
- [ ] Backup database berjalan dan **pernah diuji dipulihkan**
