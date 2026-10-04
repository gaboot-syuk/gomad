# 10 — Lingkup Rilis 1 (rilis lengkap)

> Disusun dari seluruh keputusan konsep di `docs/07`, `docs/08`, `docs/09`, `docs/12`.
>
> **Status: tujuan sudah dipakemkan — RILIS LENGKAP, bukan MVP minimal.**
>
> Istilah **"MVP" dipensiunkan** supaya tidak menyesatkan. Di mana pun di dokumen ini tertulis
> "Masuk MVP", bacalah sebagai **"Masuk Rilis 1"**. Semua fitur Mad Trans + Mad Warung masuk
> Rilis 1, dan dikerjakan **bergelombang**.

---

## 1. Prinsip penyusunan

1. **Yang tidak dipakai dua kali, jangan dibangun dua kali.** Core dibangun sekali untuk semua vertical
2. **Vertical utama dapat porsi terbesar.** Mad Warung = high frequency = mesin pertumbuhan
3. **Kalau bisa nol, jadikan nol.** Fitur yang belum terbukti dibutuhkan ditunda, bukan di-*siap-siap*
4. **Blocker kepatuhan diselesaikan sebelum menulis kode** yang menyentuhnya

---

## 2. Lingkup per pilar

### GoMad Core — **WAJIB, dibangun pertama**

| Masuk MVP | Ditunda |
|---|---|
| Ledger double-entry (akun · transaksi · entri · hold) | — |
| Unified User Identity (satu orang, banyak peran) | — |
| Access: pola **Pemilik · Staff · Pihak Luar** + batas kewenangan | Peran `finance` terpisah |
| Payment & Settlement (**PPN akun terpisah**) | Kategori transaksi lanjutan |
| Settlement ke **agency** dan **grosir** | — |
| Pembayaran per pesanan lewat **payment gateway** | Dompet/saldo (tidak dipakai sama sekali) |
| Notification (WhatsApp + in-app) | Email & push |
| Idempotency di semua callback | — |

### Mad Warung — **vertical utama**

| Masuk MVP | Ditunda |
|---|---|
| **Portal Pemilik Warung**: multi-outlet · laporan per outlet & gabungan | Analitik & prediksi |
| **Plafon penjaga + approval** | Plafon otomatis berbasis pola |
| **Jadwal shift** (absensi & akuntabilitas) | Integrasi absensi fisik |
| **Aplikasi Penjaga**: katalog · pesan · riwayat | Fitur penjaga lanjutan |
| **Mad Grosir Portal**: katalog · terima pesanan · tandai siap | Self-service katalog penuh (lihat blocker) |
| Alur order lengkap (plafon → approval → grosir → siap → antar/ambil → **bayar via gateway**) | Penjadwalan pengiriman |
| Fee: platform 3% + **service fee bulanan** | Tier fee per grosir |

### Mad Trans — ⭐ **DIPUTUSKAN: semua fitur masuk**

**Semua fitur Mad Trans masuk MVP** — travel · rental · charter · transfer penumpang · multi-stop · OTS · deposit penyewa · promo · referral · level & misi.

Yang tetap di luar hanya yang butuh **pihak lain** (bukan soal keputusan): asuransi (mitra & izin) · monitoring kendaraan (IoT) · corporate/event · premium agency · iklan · analitik.

Karena semuanya masuk, pengerjaannya **diurutkan bergelombang** — rincian di `docs/12-delta-mad-trans.md`:

| Gelombang | Isi |
|---|---|
| **0** | 4 blocker + keputusan tersisa |
| **1 · Core** | Ledger · Identity · Access · Settlement · Payment · Notification |
| **2 · Mad Warung inti** | Portal Pemilik · Halaman Penjaga · Portal Grosir |
| **3 · Mad Trans inti** | Travel + dompet agency + pencairan + COD & deposit gating |
| **4 · Mad Trans lanjutan** | Rental · Charter · multi-stop per segmen · transfer penumpang · OTS · deposit penyewa |
| **5 · Keterlibatan** | Promo · Referral · Level & Misi |
| **6 · Penutup** | Rilis store · status pengiriman |

**Catatan:** Mad Trans sudah punya produk **live**, jadi yang dibawa adalah fitur yang sudah ada — bukan dirancang dari nol.

### Mad Logistics — ⭐ **DIPUTUSKAN: nol sebagai vertical**

| Alasan |
|---|
| Pengiriman sudah ditangani **grosir** (*Supplier Self-Delivery*) atau **diambil warung sendiri** |
| Titik serah-terima sudah **dihapus** dari rencana |
| Tidak ada supply bersama dengan Mad Trans (armada idle tidak bisa antar barang) |

**Yang tetap wajib di dalam Mad Warung:**

```
Pesanan dibuat → Grosir siapkan → Siap diambil/diantar → Penjaga konfirmasi terima
```

**Kenapa konfirmasi penerimaan tidak bisa dilewatkan:** GoMad **membayar grosir lebih dulu**. Tanpa bukti barang diterima, GoMad menanggung risiko tanpa bukti kalau terjadi sengketa.

Yang ditunda: OTP, foto, GPS, rute, penugasan kurir, tracking kurir.

> **Konsekuensi:** satu vertical lebih sedikit di MVP — tapi **bukan nol fungsional**. Langkah konfirmasi penerimaan tetap ada di dalam Mad Warung.
>
> **Catatan keputusan:** Mad Logistics **tetap ada di roadmap & materi** sebagai vertical ketiga. Yang tidak dibangun di MVP hanyalah aplikasi kurir, penugasan, rute, dan tracking — bukan konsepnya.

---

## 3. ⛔ Blocker yang harus selesai sebelum menulis kode

| # | Blocker | Kenapa menghambat | Arah |
|---|---|---|---|
| ~~1~~ | ~~**Kepatuhan float**~~ ✅ **SELESAI** — warung tidak menyimpan saldo, bayar via payment gateway. Tidak ada dana pihak lain yang ditahan | — | — |
| 2 | **Siapa memelihara katalog & harga grosir** | Ratusan SKU × banyak grosir. Ini beban operasional terbesar | Pilih: grosir sendiri · GoMad · hibrida |
| 3 | **Wilayah percontohan pertama** | *Local Density Priority* butuh satu wilayah, bukan "se-Indonesia" | Pilih satu koridor (mis. Jabodetabek atau Jawa Timur) |
| 4 | **Urutan rilis: Mad Warung atau Mad Trans dulu** | Menentukan apa yang dibangun di 90 hari pertama | Lihat bagian 4 |

---

## 4. Pertanyaan strategis yang belum dijawab

> **Mad Warung adalah vertical *utama*, tapi Mad Trans sudah punya produk live.**

Dua kemungkinan, dan konsekuensinya besar:

| | Rilis Mad Trans dulu | Rilis Mad Warung dulu |
|---|---|---|
| Kecepatan ke pasar | **Cepat** — kode sudah ada, tinggal pindah ke Core baru | Lambat — dibangun dari nol |
| Bukti bahwa Core bekerja | Terbukti cepat | Terbukti setelah kerja panjang |
| Fokus tim | Terpecah | Terfokus |
| Cerita ke investor | "Kami sudah jalan" | "Kami sedang membangun" |

**Dugaan saya:** bangun **Core + Mad Warung**, tapi **rilis Mad Trans lebih dulu** begitu Core stabil — karena Mad Trans memvalidasi Core dengan risiko paling kecil. Ini butuh jawabanmu.

---

## 5. Usulan urutan 90 hari

| Fase | Isi | Keluaran |
|---|---|---|
| **0** | Selesaikan 4 blocker · kunci Mad Trans delta | Konsep utuh, siap dibangun |
| **1** | **Core**: ledger · identity · access | Pondasi yang tidak akan dibongkar |
| **2** | **Mad Trans dipindah ke Core baru** | Membuktikan Core bekerja, dengan risiko paling kecil |
| **3** | **Mad Warung**: Portal Pemilik · Aplikasi Penjaga · Mad Grosir Portal | Vertical utama hidup |
| **4** | Uji coba di **satu wilayah** | Data nyata untuk keputusan lanjutan |

**Alasan urutan ini:** Core dulu karena semua bergantung padanya · Mad Trans kedua karena paling murah untuk membuktikan Core · Mad Warung ketiga karena paling besar nilainya tapi paling besar kerjanya.

Kalau kamu lebih suka Mad Warung lebih dulu, urutannya tinggal ditukar — tapi risikonya lebih tinggi karena Core belum teruji.

---

## 6. Yang secara sadar TIDAK dibangun di MVP

| Item | Alasan |
|---|---|
| Mad Logistics sebagai vertical | Pengiriman ditangani grosir atau diambil warung |
| Titik serah-terima warung | Tidak masuk rencana awal |
| POS warung (katalog & stok internal warung) | Diganti konsep baru |
| Bayar tunai di warung (payment agent) | Tidak ada di MVP |
| Koin COD | Dihapus total |
| Akun driver & pemilik armada di ledger | Fee driver diurus internal agency |
| Multi-agency driver | Menyusul saat skala |
| Level & Misi, Referral | Bisa menyusul setelah alur inti stabil |
