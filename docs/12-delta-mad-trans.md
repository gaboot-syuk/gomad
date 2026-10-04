# 12 — Delta Mad Trans: Apa yang Dibawa, Ditunda, Dibuang

> Sumber: `promotion/08-features-future.md` (inventaris fitur 2026-09-04) + ekstraksi kode di `docs/legacy/`
> **Status: usulan untuk dikoreksi**

**Konteks:** Mad Trans sudah punya produk **live**. Jadi ini bukan "bangun apa", tapi **"apa yang dibawa"**. Semakin banyak yang dibawa, semakin besar MVP — tapi semakin sedikit yang hilang.

**Legenda:** ✅ Bawa · ⚠️ Bawa tapi **wajib diperbaiki** · ⏸️ Tunda · ❌ Buang

---

## 1. Travel — inti, sudah live

| Fitur | Putusan | Catatan |
|---|---|---|
| Cari jadwal (rute & tanggal) | ✅ | Alur inti |
| Pilih kursi (seat-map, kursi supir terkunci) | ✅ | Pembeda dari pesan via WA |
| Booking **door-to-door** (alamat jemput & antar) | ✅ | Nilai utama Mad Trans |
| Multi-penumpang | ✅ | |
| E-ticket digital + riwayat | ✅ | |
| Kelas **Ekonomi & Premium** | ✅ | |
| Kelas **Charter** | ⚠️ | Ada di legacy sebagai `travel_class` (harga flat per mobil). **Perlu keputusan** |
| Jadwal **multi-stop** | ⚠️ | Ada, **tapi cacat**: kapasitas dihitung per jadwal, bukan per segmen → penumpang A→B dan C→D saling makan kursi. **Perbaiki modelnya atau tunda** |
| Jadwal **pergi-pulang (PP)** | ✅ | Hati-hati: hold COD per **keberangkatan**, bukan per rute |
| **Approval jadwal** (supir ajukan → agency setujui) | ✅ | Bagus untuk kendali agency |
| **Transfer penumpang** antar jadwal | ⚠️ | Rumit, dan di legacy biaya transfernya selalu **0** (tidak berfungsi). **Perlu keputusan** |
| Pembatalan & kebijakan biaya | ⚠️ | Kebijakannya belum pernah ditulis di legacy |

---

## 2. Rental — sudah live

| Fitur | Putusan | Catatan |
|---|---|---|
| Kalender ketersediaan kendaraan | ✅ | Mencegah dobel-booking |
| **Lepas kunci** (self-drive) | ⚠️ | Butuh verifikasi KTP/SIM/selfie + admin. Risiko kerusakan |
| **Dengan supir** | ✅ | Lebih sederhana |
| Durasi per hari / per jam | ✅ | |
| **OTS** (bayar di tempat) | ⚠️ | Risikonya **mirip COD** — perlu perlakuan sama (gate) atau ditunda |
| **Deposit penyewa** | ⚠️ | Di legacy **tidak pernah ditagih, ditahan, maupun dikembalikan** — kolom hidup, logika mati. **Perbaiki atau buang** |
| Serah terima dokumen | ✅ | Minimal |
| Inspeksi kendaraan | ❌ | Tidak ada di legacy. Butuh kerja tambahan besar — tunda |

---

## 3. Uang & risiko

| Fitur | Putusan | Catatan |
|---|---|---|
| **COD + deposit gating** | ✅ | Sudah dirancang di `docs/02` & `docs/05` |
| Dompet agency + **pencairan** | ✅ | Dompet mitra tetap ada |
| **PPN** | ⚠️ | **Fitur baru** — tidak ada sama sekali di legacy |
| **Refund** | ⚠️ | Legacy punya status yang **menggantung tanpa pemroses**. Wajib punya jalur keluar |
| Komisi warung 2% | ❌ | Ikut dibuang bersama payment agent |
| **Koin COD** | ❌ | Dihapus total (keputusan) |

---

## 4. Keterlibatan pengguna

| Fitur | Putusan | Catatan |
|---|---|---|
| **Promo** | ⚠️ | Diskon di legacy **ditanggung agency** padahal kolomnya bilang platform. Perbaiki `cost_bearer` |
| **Referral** | ⚠️ | Sama seperti promo: dibuat `cost_bearer = platform` tapi tetap memotong agency |
| **Level & Misi (XP)** | ⏸️ | Bagus untuk retensi, tapi bukan inti alur. Menyusul setelah transaksi stabil |

---

## 5. Aplikasi & kanal

| Fitur | Putusan | Catatan |
|---|---|---|
| **Aplikasi Driver** | ✅ | Sudah ada di legacy (mulai trip, daftar penumpang, konfirmasi COD) |
| **Aplikasi Customer** | ✅ | Sudah ada |
| Rilis ke **Play Store / App Store** | ✅ | Membuka akses publik |
| **Admin console** | ✅ | Verifikasi, kelola jadwal/promo, settlement, policy |
| Onboarding agency lapangan | ✅ | Operasional, bukan fitur — tapi menentukan GTM |
| **Legal & registrasi entitas** | ⚠️ | Di legacy berstatus "sedang disiapkan". Ini **blocker**, bukan fitur |
| Aplikasi **Admin mobile** | ⏸️ | |

---

## 6. Dibuang (❌)

| Item | Alasan |
|---|---|
| Warung GoMad (customer bayar di warung) | Keputusan: tidak ada di MVP |
| Warung POS | Diganti konsep Mad Warung baru |
| Koin COD | Dihapus total |
| Komisi warung 2% | Ikut payment agent |
| Inspeksi kendaraan rental | Belum ada, butuh kerja besar |

---

## 7. Ditunda (⏸️) — dari daftar RENCANA di legacy

| Item | Alasan menunda |
|---|---|
| Premium / Featured / Subscription agency | Model pendapatan tambahan, bukan inti transaksi |
| Iklan & kerjasama brand | Butuh traffic dulu |
| Asuransi perjalanan | Butuh mitra & izin |
| Corporate & event | Segmen khusus |
| Rental lanjutan (monitoring kendaraan, paket durasi) | Butuh perangkat/IoT |
| Analitik rute & okupansi | Butuh data dulu |
| Fee passthrough gateway & convenience fee | Optimasi biaya, bukan fitur |
| Level & Misi (XP) | Retensi, bukan inti |

---

## 8. ⭐ KEPUTUSAN: **semuanya masuk MVP**

**Semua fitur Mad Trans masuk MVP** — travel, rental, charter, transfer penumpang, multi-stop, OTS, deposit penyewa, promo, referral, level & misi.

Yang **tetap di luar** hanya item yang bukan soal keputusan, melainkan **ketersediaan pihak lain**:

| Item | Kenapa tetap di luar |
|---|---|
| Asuransi perjalanan | Butuh mitra asuransi & izin |
| Monitoring kendaraan | Butuh perangkat IoT |
| Corporate & event | Butuh kesepakatan & alur khusus |
| Premium agency · iklan · analitik · fee passthrough | Bukan fitur transaksi — menyusul |

---

## 9. Konsekuensi & cara mengerjakan

**Yang berubah:** MVP berpindah dari *rilis belajar* menjadi *rilis lengkap*.

Itu keputusan yang sah. Tapi perlu disadari: **90 hari untuk seluruh lingkup ini tidak realistis**, karena yang masuk mencakup sekitar **7 permukaan aplikasi**:

```
Core · Portal Pemilik Warung · Aplikasi Penjaga · Portal Grosir
· Aplikasi Customer · Aplikasi Driver · Admin Console
```

**Solusinya bukan memotong fitur, tapi mengurutkan.** Semuanya tetap masuk MVP — hanya tidak dikerjakan serentak.

| Gelombang | Isi | Alasan urutan |
|---|---|---|
| **0** | 4 blocker + keputusan tersisa | Menentukan fondasi |
| **1 · Core** | Ledger · Identity · Access · Settlement · Payment · Notification | Semua bergantung padanya |
| **2 · Mad Trans inti** | Travel: cari jadwal · kursi · booking door-to-door · e-ticket · approval jadwal · dompet agency · pencairan · **COD + deposit gating** | Produk sudah live → memvalidasi Core dengan risiko terkecil |
| **3 · Mad Warung** | Portal Pemilik · Aplikasi Penjaga · Portal Grosir · plafon & approval · jadwal shift | Vertical utama |
| **4 · Mad Trans lanjutan** | Rental (lepas kunci & supir) · Charter · multi-stop **per segmen** · transfer penumpang · OTS · deposit penyewa | Fitur rumit; sebagian butuh perbaikan model |
| **5 · Keterlibatan** | Promo · Referral · Level & Misi | Menyentuh pembagian uang — taruh setelah alur inti stabil |
| **6 · Penutup** | Rilis Play/App Store · status pengiriman Mad Logistics | Penyelesaian |

**Kenapa Mad Trans inti di gelombang 2, bukan Mad Warung?** Karena Mad Trans sudah punya produk live — memindahkannya menguji Core dengan risiko paling kecil. Kalau Core bermasalah, ketahuan lebih awal, sebelum vertical utama dipertarungkan. *(Urutan ini bisa ditukar kalau kamu ingin Mad Warung lebih dulu.)*
