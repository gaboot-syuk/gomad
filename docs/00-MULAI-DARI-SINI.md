# 00 — MULAI DARI SINI

> **Titik masuk untuk setiap sesi baru.** Baca dokumen ini lebih dulu, lalu ikuti urutannya.
> Jika kamu membuka sesi eksekusi: baca bagian 1–4, lalu langsung ke `blueprint/06-rencana-fase.md`.

---

## 1. Apa proyek ini

**GoMad 3.0** — platform **vertical business** dengan satu Core bersama.

| Pilar | Posisi | Catatan |
|---|---|---|
| **Mad Warung** | Vertical **utama** (high frequency) | Marketplace warung ↔ grosir |
| **Mad Trans** | Vertical kedua | Travel · rental · charter (produk sudah live di `gomad.id`) |
| **Mad Logistics** | **Nol sebagai vertical** di Rilis 1 | Perannya jadi *fulfillment status* di Mad Warung |
| **GoMad Core** | Fondasi bersama | Ledger · Identity · Access · Settlement · Payment · Notification |

**Tujuan: RILIS LENGKAP** — bukan MVP minimal. Istilah "MVP" sudah dipensiunkan.

**Tidak ada tanggal**, tapi setiap fase harus **lengkap & production-ready** — bukan prototipe yang diperbaiki nanti.

---

## 2. Urutan bacaan

| # | Dokumen | Isi |
|---|---|---|
| 1 | **`00-MULAI-DARI-SINI.md`** | Dokumen ini — titik masuk & status |
| 2 | `11-konsep-gomad-v1.md` | Ringkasan induk konsep |
| 3 | `08-konsep-mad-warung-v1.md` | Konsep Mad Warung |
| 4 | `09-rantai-nilai-antar-vertical.md` | Pola bersama antar vertical |
| 5 | `05-desain-gomad-core.md` | Desain Core (ledger, fee, settlement) |
| 6 | `10-lingkup-mvp.md` | Lingkup Rilis 1 *(nama berkas warisan)* |
| 7 | `12-delta-mad-trans.md` | Apa yang dibawa dari Mad Trans |
| 8 | `13-rekomendasi-jadwal-dan-cara-kerja.md` | Cara kerja & praktik terbaik |
| 9 | **`blueprint/**`** | Blueprint teknis & rencana fase ← **untuk eksekusi** |
| 10 | `04-checklist-jangan-diulang.md` | Kriteria lulus dari kesalahan sistem lama |
| 11 | `legacy/extract-*.md` | Aturan bisnis hasil ekstraksi kode lama |

---

## 3. Aturan kerja

1. **Setiap fase harus production-ready.** Tes, penanganan error, keamanan, dan kesiapan deploy termasuk definisi selesai — bukan pekerjaan tambahan nanti
2. **Ledger disentuh lebih dulu, dan diuji lebih dulu.** Kalau salah, semua di atasnya salah
3. **Jangan buka tingkat berikutnya sebelum yang sebelumnya benar-benar selesai.** Tingkat: *demo-able* → *pilot-able* → *release-able*
4. **Kesiapan mobile dijaga di setiap fase** — lihat `blueprint/01-arsitektur.md`
5. **Uang selalu bisa ditelusuri.** Setiap mutasi punya kategori, referensi, dan jejak
6. **Tidak ada kolom atau config mati.** Kalau tidak dipakai, dihapus
7. ⭐ **Setiap fase ditutup dengan verifikasi live.** Aplikasi dijalankan lewat Docker lalu **dituturkan di browser preview editor** — bukan hanya lulus tes otomatis. Untuk mobile: **build dulu**, baru verifikasi live. Lihat `blueprint/09-lingkungan-dan-verifikasi-live.md`

---

## 4. Di mana kodenya

**Dua repo terpisah** di workspace ini:

| Repo | Isi | Teknologi |
|---|---|---|
| `api/` | Backend + web (semua peran) | Laravel + Inertia + Vue |
| `mobile/` | Aplikasi native (menyusul) | Flutter |

| Lokasi | Isi |
|---|---|
| `docs/` | **Sumber kebenaran** — blueprint, keputusan, rencana fase |

> **Konsekuensi dua repo:** `docs/` adalah penghubung netral. **Kontrak API** (`blueprint/03-kontrak-api.md`)
> menjadi satu-satunya antarmuka antara `api/` dan `mobile/` — dia yang menjaga keduanya tetap sinkron.

---

## 5. Status saat ini

| Fase | Status |
|---|---|
| **Konsep** | ✅ Selesai — terkunci di `docs/08` · `09` · `11` |
| **Blueprint** | ✅ **Selesai — 9 dari 9** · indeks di `blueprint/README.md` |
| **Fase 0** | 🔄 Dikerjakan — stack lokal, verifikasi live, backup/restore lokal, dan kontrak v0 siap; staging dan integrasi remote menunggu konfigurasi |
| **Fase 1–9** | ⬜ Belum mulai — mulai setelah seluruh kriteria Fase 0 lulus |

### Keputusan yang sudah selesai · 2026-10-04

| # | Item | Keputusan |
|---|---|---|
| 1 | **PPN** | **Ditambahkan ke pembeli** — customer bayar harga + fee + PPN |
| 2 | **Katalog grosir** | **Hibrida** — GoMad isi katalog awal, grosir yang memperbarui harga & stok |
| 3 | **Fallback pembayaran** | Metode default (token) + **fallback QRIS/VA** |
| 4 | **Kebijakan pembatalan** | Disetujui: >24 jam refund penuh · ≤24 jam potong 25% · setelah berangkat tidak ada |
| 5 | **Wilayah** | Data referensi **seluruh Indonesia** sejak awal; fokus rilis **Jabodetabek** (sasaran luas: Jabodetabek · Surabaya · Solo–Yogya) |
| 6 | **Urutan rilis** | Core → **Mad Warung** → Mad Trans → pendukung |
| 7 | **Service fee Mad Warung** | **Rp 0** di Rilis 1 — hanya platform fee 3% + PPN |
| 8 | **Plafon penjaga** | Diturunkan dari volume warung (1,5× per pesanan · 4× harian); fallback Rp 500rb / Rp 1,5jt |
| 9 | **Settlement grosir** | **Real-time saat penjaga konfirmasi terima** — bukan siklus |
| 10 | **Prefix kode** | Dipisah per vertical: `MW-` `MT-` `WD-` `ST-` `RF-` |
| 11 | **Di luar Rilis 1** | Asuransi · monitoring kendaraan · corporate/event · premium agency · warung premium · iklan · skala B2B→B2B2C · analitik |

### Masih menunggu keputusan

| # | Item |
|---|---|
| — | **Tidak ada yang memblokir.** Seluruh keputusan pembuka sudah selesai |

> Sisanya adalah penyesuaian saat berjalan: angka plafon per warung diisi saat onboarding, dan wilayah operasional kedua dibuka setelah Jabodetabek terbukti.

---

## 6. ⭐ Protokol handoff — wajib di akhir setiap sesi

Supaya ganti sesi aman, **tiga hal ini diperbarui setiap sesi berakhir**:

| # | Yang diperbarui | Di mana |
|---|---|---|
| 1 | **Pelacak progres** — fase mana selesai, apa yang sedang dikerjakan, apa berikutnya | `blueprint/06-rencana-fase.md` bagian *Status* |
| 2 | **Keputusan baru** — apa pun yang diputuskan di sesi itu | Dokumen konsep terkait di `docs/`, lalu diringkas ke `/memories/repo/gomad.md` |
| 3 | **Ringkasan orientasi** — kondisi terkini dalam beberapa baris | `/memories/repo/gomad.md` |

**Aturan penting:** jangan taruh apa pun yang penting di `/memories/session/` — isinya dihapus setelah sesi berakhir. Yang permanen adalah **`docs/`**.

**Cara memulai sesi baru** — tempel kalimat ini:

> **Baca `docs/00-MULAI-DARI-SINI.md`, lalu mulai Fase 0 dari `docs/blueprint/06-rencana-fase.md`.**
> Kerjakan langkahnya berurutan, dan tutup fase dengan verifikasi live di browser preview.

Sesi berikutnya akan langsung tahu: apa proyeknya, urutan bacaan, di mana kodenya, aturan kerjanya, dan langkah pertama yang harus dikerjakan.
