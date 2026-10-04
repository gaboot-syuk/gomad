# P2 — Bagus kalau ada

> Opsional. Tidak memblokir Tahap 1, tapi menaikkan kualitas keputusan.

---

## P2-1 · Wireframe, Figma, atau PRD lama

**Kenapa berguna:** mempercepat pemahaman niat desain, terutama alur yang tidak jelas dari kode (mis. kenapa checkout punya langkah tertentu).

**Yang ditemukan di repo:**
- ⚠️ **Tidak ada file Figma/PRD.** Yang ada penggantinya: `demo-static/` — ±258 halaman HTML mockup (`demo.gomad.id`) yang cukup lengkap per role. Ini bisa dipakai sebagai wireframe de-facto.
- `gomad-docs/flow-shots/` — tangkapan layar diagram alur.

**Catatan:** mockup di `demo-static/` adalah **tampilan demo**, bukan cerminan perilaku sistem. Jangan dipakai sebagai sumber kebenaran logika.

---

## P2-2 · Repo backend saja

**Kenapa berguna:** kalau hanya butuh bagian tertentu, kirim sebagian saja.

**Yang minimal dibutuhkan (kalau tidak mau kirim semuanya):**
1. `gomadid/database/migrations/` — bentuk data
2. `gomadid/app/Services/` — aturan bisnis
3. `gomadid/app/Enums/` — status & lifecycle
4. `gomadid/config/gomad.php` — parameter bisnis
5. `gomadid/routes/` — permukaan API

**Status di repo ini:** ✅ **seluruh backend sudah ada di repo ini** (`gomadid/`), jadi item ini praktis sudah selesai — tinggal dirapikan sesuai daftar di atas.

**Catatan:** ada juga `gomadid-back/` (backup/staging, DB lokal). Perlu dikonfirmasi apakah masih dipakai sebelum dijadikan referensi.

---

## P2-3 · Data analytics/penggunaan

**Kenapa berguna:** memisahkan fitur yang benar-benar dipakai dari fitur yang mati — menghemat usaha rebuild, dan mencegah "memindahkan sampah".

**Yang dibutuhkan minimal:**
- Jumlah pengguna aktif per role
- Booking per minggu/bulan (tren)
- Distribusi metode pembayaran (digital vs COD vs OTS) — ini yang menentukan seberapa besar `Core/Settlement` harus dibangun
- Tingkat pembatalan & kedaluwarsa
- Retensi agency

**Yang ditemukan di repo:** ⚠️ **Tidak ada.** Tidak ada folder analytics, tidak ada ekspor laporan. Beberapa angka di `gomad-docs/investor/` bersifat proyeksi, bukan data aktual — **jangan tertukar**.

**Kalau tidak ada:** jumlah baris per tabel (P1-12) bisa jadi pendekatan kasar yang cukup berguna.

---

## P2-4 · Catatan awal dari investor soal Mad Warung

Sama seperti yang aku paparkan tadi.