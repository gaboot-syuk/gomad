# Blueprint — Indeks

> Blueprint teknis lengkap untuk eksekusi GoMad 3.0.
> Setiap dokumen punya cakupan jelas. Status ditandai agar sesi berikutnya tahu apa yang harus disusun.

| # | Dokumen | Status | Cakupan |
|---|---|---|---|
| 01 | `01-arsitektur.md` | ✅ **Selesai** | Struktur repo · batas modul · aturan dependensi · pola peran (Pemilik/Staff/Pihak Luar) · auth ganda (session + token) · **aturan kesiapan mobile** · konvensi kode |
| 02 | `02-model-data-ledger.md` | ✅ **Selesai** | Skema tabel · entitas identitas & multi-outlet · `ledger_accounts/transactions/entries/holds` · bagan akun · relasi antar domain |
| 03 | `03-kontrak-api.md` | ✅ **Selesai** | Konvensi API · versi · format error tunggal · autentikasi · daftar endpoint per vertical · **cara generate klien Dart** untuk mobile |
| 04 | `04-layar-per-peran.md` | ✅ **Selesai** | Daftar halaman per peran · alur tiap peran · keadaan kosong/error/loading · aturan responsif |
| 05 | `05-aturan-bisnis.md` | ✅ **Selesai** | Fee & PPN · deposit gating · plafon & approval · jadwal shift · settlement per mitra · refund · promo `cost_bearer` · transfer penumpang · multi-stop per segmen |
| 06 | `06-rencana-fase.md` | ✅ **Selesai** | 10 fase · kriteria lulus · kesiapan mobile · pelacak status |
| 07 | `07-strategi-uji.md` | ✅ **Selesai** | Uji wajib alur uang · rekonsiliasi · idempotency · race condition · uji beban ringan · uji pemulihan backup |
| 08 | `08-deployment.md` | ✅ **Selesai** | Lingkungan lokal/staging/produksi · CI · backup & pemulihan · monitoring & alerting · rahasia & konfigurasi · prosedur rilis |
| 09 | `09-lingkungan-dan-verifikasi-live.md` | ✅ **Selesai** | Stack Docker · port · perintah harian · **protokol verifikasi live web & mobile** · batas jujur verifikasi live |

---

## Prinsip yang berlaku untuk semua dokumen

1. **Konsisten dengan keputusan yang sudah dikunci** di `docs/08` · `09` · `11` · `12`. Kalau ada yang berbeda, jelaskan alasannya dan perbarui dokumen sumbernya
2. **Kesiapan mobile dijaga**, bukan ditambahkan belakangan
3. **Tidak ada kolom, config, atau endpoint mati.** Kalau tidak dipakai, tidak ditulis
4. **Setiap aturan uang harus bisa diuji** — kalau tidak bisa diuji, desainnya belum selesai

## Cara memakai

- **Sesi penyusunan blueprint**: kerjakan berurutan 01 → 08
- **Sesi eksekusi**: baca `06-rencana-fase.md`, kerjakan fase yang tertulis di *Pelacak Status*
- Setelah dokumen selesai, ubah statusnya di tabel di atas
