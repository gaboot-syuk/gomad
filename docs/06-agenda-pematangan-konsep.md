# 06 — Agenda Pematangan Konsep GoMad

> **Status:** eksekusi dihentikan atas permintaan. Skeleton `apps/api` sudah dibatalkan & dihapus.
> **Tujuan dokumen ini:** memetakan apa yang **sudah matang**, apa yang **belum**, dan urutan mematangkannya.

---

## 1. Temuan yang menentukan

Saya menyisir seluruh `gomad-docs/` (17 dokumen investor + 8 paket promosi + arsip).

**Hasil pencarian kata kunci:**

| Kata kunci | Ditemukan di |
|---|---|
| `Mad Warung` | ❌ **nol** — hanya di catatan handover milikmu sendiri |
| `grosir` / `wholesale` | ❌ **nol** |
| `Mad Logistics` | ❌ **nol** |
| `GoMad Core` | ❌ **nol** |
| `vertical business` | ❌ **nol** |

**Kesimpulan: konsep baru GoMad tidak terdokumentasi sama sekali.**

Yang kita punya saat ini:

| Artefak | Isinya |
|---|---|
| `mindmap.png` | Konsep baru — tapi hanya bagan, tanpa penjelasan |
| `gomad-docs/**` | Konsep **LAMA** — produk satu-vertical (Mad Trans), bukan vertical business |
| `docs/**` (buatan kita) | Turunan dari kode lama + keputusan teknis sesi ini |
| Percakapan ini | Tempat konsep baru benar-benar hidup |

Artinya: **konsep GoMad 3.0 baru ada di kepalamu dan di percakapan.** Itu risiko terbesar proyek ini — bukan masalah teknis, tapi masalah bahwa blueprint-nya belum dituliskan. Semua keputusan teknis yang kita ambil (ledger, deposit, fee) adalah fondasi yang benar, tapi konsep di atasnya masih kosong.

---

## 2. Status kematangan per pilar

| Pilar | Status | Catatan |
|---|---|---|
| **Business Strategy** | 🟢 Cukup | Visi, prinsip asset-light, local density, North Star, roadmap 90d/6m/12m — dari mindmap |
| **GoMad Core** | 🟢 Cukup | Ledger, fee, settlement, deposit gating, idempotency — sudah mengeras di `docs/05` |
| **Mad Trans** | 🟡 Sebagian | Domain-nya terbaca dari kode lama, tapi **belum diputuskan apa yang berubah** di konsep baru |
| **Mad Logistics** | 🔴 Belum | Baru sebatas "minimal, fulfillment Mad Warung" — tanpa model |
| **Mad Warung** | 🔴 **Belum** | **Celah terbesar.** Konsepnya belum pernah dituliskan di mana pun |
| **Rantai nilai antar pilar** | 🔴 Belum | Belum jelas apa yang benar-benar menghubungkan ketiga vertical |
| **Lingkup MVP** | 🔴 Belum | "MVP GoMad keseluruhan" belum bisa jadi lingkup 90 hari |

---

## 3. Celah konsep — berurutan sesuai ketergantungan

### Celah 1 · Mad Warung — konsep bisnisnya sendiri

Ini yang paling mendasar. Pertanyaan yang harus terjawab sebelum satu baris kode:

**Masalah & pelaku**
- Masalah apa yang dipecahkan, dari sudut pandang siapa — warung, grosir, atau keduanya?
- "Warung tidak perlu antri dan tidak perlu ambil ke grosir" — ini menghapus **dua** friksi. Yang mana yang paling menyakitkan bagi warung?
- Apakah grosir juga diuntungkan, atau ini murni memindahkan beban ke mereka?

**Rantai pasok**
- Siapa pemasoknya: grosir existing, distributor, atau **GoMad yang jadi grosir** (beli lalu jual)?
- Kalau grosir existing: apa insentif mereka ikut?
- Kalau GoMad jadi grosir: butuh modal kerja & gudang — bertentangan dengan prinsip *asset-light*?

**Alur & kanal**
- Bagaimana warung memesan: aplikasi, WhatsApp, telepon, atau **agen lapangan**?
- Bagaimana grosir menerima pesanan: dashboard, WhatsApp, atau GoMad yang input manual?
- Kalau warung tidak bisa pakai aplikasi sendiri, siapa yang membantu?

**Uang**
- Siapa menetapkan harga: grosir, GoMad, atau negosiasi?
- Sumber pendapatan GoMad: komisi dari grosir, margin dari harga, biaya layanan ke warung, atau langganan?
- Bagaimana warung membayar: transfer di muka, tunai saat kirim, atau **tempo/utang**?
- Kalau tempo — siapa menanggung risiko gagal bayar?

**Logistik & mutu**
- Siapa yang mengantar: grosir, kurir GoMad, atau pihak ketiga?
- Minimum order? Barang dikonsolidasi antar beberapa warung sekali jalan?
- Bagaimana kalau stok kurang, barang tidak sesuai, atau telat?
- Bagaimana pengembalian/komplain?

**Frekuensi (inti nilai investasi)**
- Apa yang membuat warung memesan **lagi dan lagi** — bukan sekali coba?
- Apakah harga lebih murah, lebih cepat, atau lebih mudah? Pilih satu yang jadi keunggulan utama.

### Celah 2 · Rantai nilai antar pilar

- Apa yang **benar-benar** menghubungkan Mad Warung, Mad Trans, dan Mad Logistics?
- Apakah ada satu identitas pengguna yang sama (pemilik warung juga penumpang)?
- Sinergi yang mungkin: armada travel **idle** jadi kurir? warung jadi **titik serah-terima**? rute travel searah rute pengiriman?
- **Atau** ketiganya sebenarnya berdiri sendiri dan hanya berbagi GoMad Core?
- Apakah konsumen akhir ikut masuk platform, atau berhenti di warung?

> Pertanyaan ini menentukan apakah kita membangun **satu ekosistem** atau **tiga produk** yang berbagi infrastruktur. Bedanya besar sekali.

### Celah 3 · Peran Mad Logistics

- Kalau grosir mengantar sendiri, apa kerja platform? Hanya mencatat + POD?
- Kalau platform mengantar — siapa kurirnya: mitra baru, atau armada travel yang idle?
- POD sudah didefinisikan (OTP + foto + GPS di mindmap) — tapi **siapa** yang melakukan?

### Celah 4 · Lingkup MVP

- "MVP GoMad keseluruhan" belum bisa jadi lingkup. Perlu batas tegas: fitur mana **masuk**, mana **ditunda**.
- Roadmap 90 hari itu tentang apa secara konkret?

### Celah 5 · Mad Trans — apa yang berubah dari yang sudah live

Produk lama sudah jalan dengan fitur berikut. Mana yang dipertahankan, diubah, dibuang?

- Travel (jadwal, kursi, door-to-door, multi-stop, PP) · Rental (lepas kunci, dengan supir, kalender) · Charter
- Transfer penumpang antar jadwal · Level & Misi (XP) · Referral · promo
- **Charter & Rental masuk MVP, atau menyusul?**

### Celah 6 · Identitas, peran, dan multi-user agency

- Apakah agency boleh punya banyak pengguna (operator, dispatcher, kasir)?
- Apakah peran `finance` dipisah dari `admin`?
- Satu orang boleh punya beberapa peran/perusahaan sekaligus?

### Celah 7 · Unit economics & proyeksi

- Dokumen investor memakai asumsi **produk lama**. Setelah konsep baru (3 vertical, fee baru), proyeksi perlu divalidasi ulang.
- Berapa warung & agency yang dibutuhkan agar *break even*?

### Celah 8 · Non-fungsional

- Target skala, anggaran infra bulanan, SLA, dan rencana uji coba terbatas (*local density* — Madura dulu?).

---

## 4. Urutan pematangan yang saya sarankan

| Tahap | Topik | Alasan urutan |
|---|---|---|
| **1** | **Mad Warung** (Celah 1) | Celah terbesar, paling baru, dan jadi alasan investor masuk |
| **2** | **Rantai nilai antar pilar** (Celah 2) | Menentukan apakah Mad Logistics perlu ada, dan apakah ini satu ekosistem |
| **3** | **Mad Logistics** (Celah 3) | Baru bisa dijawab setelah 1 & 2 |
| **4** | **Lingkup MVP** (Celah 4) | Batas 90 hari, disusun setelah bentuk ekosistemnya jelas |
| **5** | **Mad Trans: delta** (Celah 5) | Memangkas fitur lama yang tidak perlu dibawa |
| **6** | **Identitas & peran** (Celah 6) | Menyempurnakan Core |
| **7** | **Unit economics & proyeksi** (Celah 7) | Butuh lingkup MVP yang sudah pasti |
| **8** | **Non-fungsional** (Celah 8) | Terakhir, karena bergantung pada semuanya |

---

## 5. Cara kerja yang saya usulkan

Untuk **Celah 1 (Mad Warung)**, dokumen tidak akan muncul dari analisis kode — konsepnya belum pernah ditulis. Cara tercepat: **sesi tanya-jawab terpandu**, saya bertanya per kelompok, kamu jawab, dan saya rangkum jadi dokumen konsep yang bisa kamu koreksi.

Kalau kamu sudah punya bahan sendiri (catatan investor, hasil diskusi, atau dokumen dari pihak investor), taruh saja di workspace — saya baca dulu, supaya sesinya tidak mengulang yang sudah ada.

**Yang tidak akan saya lakukan sebelum konsepnya matang:** menulis kode, membuat skema, atau menyiapkan skeleton.

---

## 6. Ketidaksesuaian yang saya temukan (perlu keputusan)

Dari `promotion/08-features-future.md` (inventaris fitur 2026-09-04):

| Temuan | Konflik | Keputusan |
|---|---|---|
| **Biaya layanan Rental = Rp 10.000**, sedangkan Travel Rp 5.000 | Kamu menyebut "service fee flat 5.000" — apakah itu hanya Travel? | ⏳ |
| **Warung payment-agent sudah `[AKTIF]`** (login PIN, kode `WM-`, komisi 2%, settlement Senin) | Kamu bilang "tidak ada bayar di warung" untuk MVP — ini **mencabut fitur yang sudah sebagian live** | ⏳ |
| **Settlement legacy = Senin** | Kamu memilih default **Sabtu** | ✅ Sudah jelas (mengganti) |
| **Model B sudah dikunci & diimplementasikan** (sesuai `master-brief.md`) | Ini **mengonfirmasi** struktur fee — bagus, tidak perlu dibangun dari nol | ✅ Terkonfirmasi |
| **Promo ditanggung agency** (padahal kolom `cost_bearer` ada) | Belum diputuskan siapa yang menanggung | ⏳ |
