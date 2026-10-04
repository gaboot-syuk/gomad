# 11 — Konsep GoMad v1 (induk)

> **Ringkasan induk konsep GoMad 3.0 — vertical business.**
> Dokumen ini merangkum; detailnya ada di dokumen rujukan masing-masing.
> Status: **Mad Warung terkunci · Mad Trans menunggu delta · sisanya terbuka**

---

## 1. Ringkasan

**GoMad = platform vertical business** dengan satu Core bersama.

| Pilar | Posisi | Frekuensi |
|---|---|---|
| **Mad Warung** | Vertical **utama** | High — warung kulakan **min. 2× sehari** |
| **Mad Trans** | Vertical **kedua** | Low — travel/rental sesekali |
| **Mad Logistics** | **Nol di MVP** | — |
| **GoMad Core** | Fondasi bersama | — |

Alasan Mad Warung jadi utama: frekuensi tingginya selaras **North Star: Transaction Frequency**.

---

## 2. Yang benar-benar menyatukan GoMad

Bukan aliran barang atau aset — melainkan **pola yang sama, diulang di tiap vertical**.

```
PEMILIK USAHA   → punya usaha · multi-unit · pegang uang · lihat semua aktivitas
   STAFF        → akun sendiri · punya jadwal · dibatasi kewenangannya
   PIHAK LUAR   → pelanggan atau pemasok
```

| Vertical | Pemilik Usaha | Staff | Pihak Luar |
|---|---|---|---|
| **Mad Trans** | Pemilik travel/rental (agency) | **Driver** | Penumpang |
| **Mad Warung** | Pemilik warung | **Penjaga Warung** | **Grosir** |

**Temuan kunci:** *Driver* dan *Penjaga Warung* secara struktur identik. Dan **Grosir menempati posisi identik dengan Agency** — keduanya penjual yang menerima settlement dari GoMad.

Akibatnya: **mesin settlement Core dibangun sekali, dipakai dua vertical.** Itu yang membuat "vertical business dengan Core bersama" jadi nyata, bukan slogan.

Lengkapnya: `docs/09-rantai-nilai-antar-vertical.md`

---

## 3. Model uang

### Struktur fee

```
service fee   = Rp 5.000 flat (Travel)      |  bulanan (Mad Warung)
platform fee  = 3% × fare / harga barang
PPN           = 11% × (service fee + platform fee)   → akun terpisah, BUKAN pendapatan
```

Penjual menerima **harga penuh** (Model B) — platform tidak pernah memotong dari harga penjual.

### Dompet GoMad hanya satu arah ⭐

| Pihak | Punya saldo? |
|---|---|
| Penumpang, **Warung** | ❌ Tidak — bayar per transaksi via gateway |
| **Agency, Grosir** | ✅ Ya — menerima uang dari GoMad, lalu cairkan |

> **GoMad hanya menyimpan utang ke penjual, bukan uang konsumen.**
> Konsekuensi: **jauh dari ranah uang elektronik.** Blocker float hilang bukan karena dihindari, tapi karena modelnya memang tidak butuh.

### Settlement

| Penerima | Pengaturan |
|---|---|
| Agency | Diatur **per agency**, default mingguan (Sabtu) |
| Grosir | Diatur **per grosir**; GoMad mem-backup pembayaran di tahap awal |

**Self-funding:** warung sudah membayar di muka per pesanan lewat gateway, jadi mem-backup grosir tidak butuh modal tambahan.

---

## 4. GoMad Core

| Komponen | Isi |
|---|---|
| **Ledger** | Double-entry · berbasis akun · hold sebagai *record* |
| **Identity** | Unified User Identity — satu orang, banyak peran |
| **Access** | Pola Pemilik · Staff · Pihak Luar + batas kewenangan |
| **Settlement** | Untuk agency & grosir, PPN akun terpisah |
| **Notification** | WhatsApp + in-app |

**Akun**: `PLATFORM.*` · `AGENCY.AVAILABLE` · `AGENCY.COLLATERAL` · `GROSIR.AVAILABLE`
**Tanpa akun**: warung, penumpang.

Detail: `docs/05-desain-gomad-core.md`

---

## 5. Mad Warung — vertical utama

**Marketplace** antara Grosir dan Warung retail. Asset-light.

**Menghapus:** antri & keliling. **Keunggulan sesungguhnya: waktu** — warung 24 jam tidak perlu ditinggal. Celah harga hanya 5%, jadi harga adalah **syarat minimum**, bukan keunggulan.

**Struktur tiga lapis:** Pemilik (bisa banyak outlet) → Warung → Penjaga (akun sendiri + jadwal shift).

**Posisi:** dari *alat belanja* menjadi **alat pengawasan usaha** — pemilik mengawasi uang & stok yang lewat tangan penjaga.

**Kendali:** plafon penjaga + approval (wajib bisa dari jauh).

**Pembayaran:** penjaga menekan tombol bayar; dana dari metode pembayaran milik pemilik.

Detail: `docs/08-konsep-mad-warung-v1.md`

---

## 6. Mad Trans — vertical kedua · ⭐ **DIPUTUSKAN**

**Keputusan: SEMUA fitur Mad Trans masuk MVP** — travel, rental, charter, transfer penumpang, multi-stop, OTS, deposit penyewa, promo, referral, level & misi.

Domainnya sudah terbaca dari produk **live** di `gomad.id`, jadi yang dibawa adalah fitur yang sudah ada — bukan dirancang dari nol.

**Yang tetap di luar** hanya yang butuh pihak lain (bukan soal keputusan): asuransi perjalanan · monitoring kendaraan (IoT) · corporate/event · premium agency · iklan · analitik.

Karena semuanya masuk, pengerjaannya **diurutkan jadi gelombang** — lihat `docs/12-delta-mad-trans.md`.

---

## 7. Mad Logistics · ⭐ **DIPUTUSKAN**

**Keputusan:** **nol sebagai vertical di MVP** — perannya dibatasi menjadi *fulfillment status* di dalam Mad Warung.
**Mad Logistics tetap ada di roadmap dan materi** sebagai vertical ketiga, jadi cerita "tiga vertical" tetap utuh tanpa harus membangun aplikasi kurir.

**Alasan: tidak ada kekosongan yang perlu diisi.**

| # | Alasan |
|---|---|
| 1 | Pengiriman sudah ditangani **grosir** (punya salesman) atau **diambil warung sendiri** |
| 2 | **Titik serah-terima sudah dihapus** dari rencana |
| 3 | **Tidak ada supply yang bisa dipakai bersama** — armada travel idle tidak bisa antar barang. Kalau Mad Logistics dibangun, dia butuh supply baru dari nol |
| 4 | Seluruh elemen mindmap-nya sudah terpenuhi tanpa GoMad: *Supplier Self-Delivery* = jalur grosir · *Mad Warung Fulfillment* = status pesanan · *Partner Delivery Network* = "nanti" · *Batched & Scheduled Routes* & *Proof of Delivery (OTP/foto/GPS)* tidak diperlukan karena tidak ada armada GoMad |

### ⚠️ Tapi "nol" bukan berarti tidak ada apa-apa

Satu hal **tetap wajib**: **konfirmasi penerimaan oleh penjaga.**

Alasannya: GoMad **membayar grosir lebih dulu** (di-backup). Tanpa bukti barang diterima, GoMad menanggung risiko **tanpa bukti** — kalau ada sengketa ("barang tidak sampai"), posisinya lemah.

```
Pesanan dibuat → Grosir siapkan → Siap diambil/diantar → Penjaga konfirmasi terima
                                                              ↑
                                                    ini yang wajib ada
```

Yang ditunda: OTP, foto, GPS, rute, penugasan kurir. Yang tetap ada: **satu langkah konfirmasi** di dalam Mad Warung.

### Kapan Mad Logistics perlu dinyalakan

| Pemicu |
|---|
| Grosir tidak sanggup mengantar ke wilayah tertentu |
| Volume cukup untuk **konsolidasi** — satu kendaraan antar ke banyak warung sekaligus |
| Butuh pengiriman **antar-warung** (belum ada di konsep) |
| GoMad ingin **kendali mutu penuh** atas ketepatan waktu |

Jadi Mad Logistics **ditunda sampai ada pemicunya**, bukan dibuang permanen.

---

## 8. Lingkup Rilis 1 (rilis lengkap)

| Pilar | Lingkup |
|---|---|
| **Core** | Ledger · Identity · Access · Settlement · Payment · Notification |
| **Mad Warung** | Portal Pemilik · Aplikasi Penjaga · Mad Grosir Portal · plafon & approval · jadwal shift |
| **Mad Trans** | ✅ Diputuskan: **semua fitur masuk** — dikerjakan bergelombang (`docs/12`) |
| **Mad Logistics** | ✅ Diputuskan: nol sebagai vertical — perannya jadi *fulfillment status* di Mad Warung, tetap ada di roadmap |

**Kerangka pengerjaan (gelombang):** 0 blocker → 1 Core → 2 Mad Trans inti → 3 Mad Warung → 4 Mad Trans lanjutan → 5 Keterlibatan → 6 Penutup.

Detail: `docs/10-lingkup-mvp.md` *(nama berkas warisan — isinya Rilis 1)*

---

## 9. ⚠️ Hal yang harus diselesaikan sebelum menulis kode

| # | Item | Status |
|---|---|---|
| 1 | ~~Kepatuhan float~~ | ✅ **Selesai** — model tidak menahan dana konsumen |
| 2 | **Kartu debit & OTP/3DS** — kartu debit Indonesia umumnya minta OTP per transaksi, sehingga "penjaga tinggal klik" bisa gagal | ⛔ **Belum** — rancang metode default + fallback QRIS/VA |
| 3 | **Siapa memelihara katalog & harga grosir** (ratusan SKU) | ⛔ Belum |
| 4 | **Wilayah percontohan pertama** | ⛔ Belum |
| 5 | **Urutan rilis: Mad Warung atau Mad Trans dulu** | ⛔ Belum |
| 6 | **Mad Trans: apa yang dibawa & dibuang** | ⛔ Belum |
| 7 | Penyimpanan data kartu | Rancangan: pakai **token gateway**, jangan simpan sendiri (PCI-DSS) |

---

## 10. Yang secara sadar TIDAK dibangun di MVP

Mad Logistics sebagai vertical · titik serah-terima · POS warung · bayar tunai di warung · Koin COD · akun driver & pemilik armada · multi-agency driver · dompet untuk pembeli

---

## 11. Daftar dokumen

| Dokumen | Isi |
|---|---|
| `docs/01-temuan-legacy.md` | Recon sistem lama & D1–D8 |
| `docs/02-keputusan-d1-d2-d3.md` | Deposit gating · struktur fee |
| `docs/03-keputusan-turunan.md` | D9–D15 |
| `docs/04-checklist-jangan-diulang.md` | Kriteria lulus sistem baru |
| `docs/05-desain-gomad-core.md` | Desain Core |
| `docs/06-agenda-pematangan-konsep.md` | Agenda & celah konsep |
| `docs/07-konsep-mad-warung.md` | Catatan proses Mad Warung |
| `docs/08-konsep-mad-warung-v1.md` | **Konsep Mad Warung (rujukan)** |
| `docs/09-rantai-nilai-antar-vertical.md` | **Rantai nilai & pola bersama** |
| `docs/10-lingkup-mvp.md` | **Lingkup Rilis 1 (rilis lengkap)** |
| `docs/11-konsep-gomad-v1.md` | **Dokumen ini — ringkasan induk** |
| `docs/legacy/extract-*.md` | Ekstraksi aturan dari kode lama |
