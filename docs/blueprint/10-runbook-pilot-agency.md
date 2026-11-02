# Runbook — Uji pilot dengan 1 agency sungguhan

**Tujuan:** membuktikan satu perjalanan Mad Trans berjalan ujung ke ujung dengan
agency nyata, bukan data demo, sebelum Fase 6 dimulai.

**Prasyarat:** stack lokal sehat (`bash bin/compose ps`) atau staging Render aktif.

---

## 1. Daftar periksa sebelum hari-H

| # | Item | Cara memastikan |
|---|---|---|
| 1 | Zona waktu benar | `APP_TIMEZONE=Asia/Jakarta` ada di `.env` (lokal) dan di Environment staging |
| 2 | Tarif terpasang | `MAD_TRANS_SERVICE_FEE`, `MAD_TRANS_PLATFORM_FEE_BASIS_POINTS`, `MAD_TRANS_PPN_BASIS_POINTS` |
| 3 | Kredensial Midtrans masuk container | `bash bin/compose exec app printenv \| grep MIDTRANS` harus menampilkan 6 baris. Jika kosong, container dibuat sebelum berkas kredensial ada → `bash bin/compose up -d --force-recreate --no-dep app queue scheduler` lalu `bash bin/compose restart nginx` (nginx menyimpan IP container lama) |
| 4 | `MIDTRANS_SNAP_URL` benar | Harus `https://app.sandbox.midtrans.com/snap/v1/transactions`. Nilai `.../snap/snap.js` (URL pustaka JS) kini ditoleransi dan dinormalkan otomatis, tetapi perbaiki sumbernya |
| 5 | Izin & bagan akun ada | `php artisan db:seed --class="App\Core\Access\Infrastructure\AccessControlSeeder" --force` dan `...LedgerAccountSeeder --force` |
| 6 | Tes hijau | `bash bin/test` → 85 tes |

## 2. Onboarding agency (data nyata)

1. Agency mendaftar lewat `/register`, lalu membuat usaha bertipe **Agency** di `/dashboard`.
2. Pemilik membuka **Portal agency** → *Tambah rute* (kota asal/tujuan, titik jemput, harga).
3. **Tambah armada** dengan kode kursi sebenarnya, mis. `A1 A2 A3 A4 B1 B2 B3 B4`.
4. Undang supir sebagai anggota dengan peran `driver`; supir menerima tautan undangan.
5. **Isi jaminan COD** lewat kartu *Jaminan COD* (Midtrans). Tanpa jaminan, booking COD ditolak —
   booking online tetap jalan.
6. Buat jadwal: pilih rute, armada, supir, waktu berangkat, harga → status **Menunggu persetujuan**.
7. **Setujui** jadwal → status **Terbit** dan muncul di pencarian penumpang.
8. **Aktifkan COD** bila agency siap menerima tunai.

## 3. Alur uji yang wajib ditelusuri

| Peran | Langkah | Bukti yang dikumpulkan |
|---|---|---|
| Penumpang | Cari rute + tanggal, pilih kursi, pesan (online **dan** COD) | Kode e-ticket, rincian biaya |
| Penumpang | Buka e-ticket, pastikan kursi sama tidak bisa dipesan ulang | Denah kursi kursi tersebut terkunci |
| Supir | Buka *Tugas saya* → **Mulai perjalanan** saat jam berangkat tiba | Status `in_progress` |
| Supir | **Konfirmasi naik** penumpang COD; **Tidak naik** untuk yang tidak datang | Waktu boarding, jaminan dilepas |
| Supir | **Selesaikan perjalanan** | Status `completed`, booking `completed` |
| Agency | Periksa daftar penumpang per jadwal | Jumlah kursi terisi, status boarding |
| Agency | Periksa saldo jaminan | Berkurang sebesar biaya layanan + platform + PPN per penumpang COD |
| Penumpang | Unggah KTP, SIM, selfie di `/mad-trans/rental-documents` | Status tiap berkas `uploaded`, berkas tidak bisa diakses publik |
| **Admin platform** | Verifikasi ketiga berkas di `/admin/rental-documents` | Status `verified`, waktu verifikasi tercatat |
| Penumpang | Ajukan **lepas kunci**; sebelum verifikasi opsi ini tidak muncul | Opsi hanya ada setelah 3 berkas `verified` |
| Agency | Buka **Dokumen penyewa** saat sewa `active` | Halaman terbuka selama sewa berjalan |
| Agency | Buka dokumen yang sama setelah sewa `completed` | **403** — akses dicabut (`documents_revoked_at`) |

## 4. Keadaan gagal yang wajib dicoba

- Booking COD saat saldo jaminan kurang → ditolak dengan pesan jelas.
- Buka jadwal sebelum jam berangkat → tombol mulai tidak muncul, ada keterangan waktunya.
- Supir lain membuka tugas yang bukan miliknya → 403.
- Agency lain membuka `/mad-trans/agency/{id}` milik orang → 403.
- Pesan kursi yang sama dari dua tab hampir bersamaan → satu berhasil, satu ditolak.
- Agency membuka `/admin/rental-documents` → 403 (verifikasi dokumen hanya untuk admin platform).
- Agency membuka dokumen penyewa sebelum sewa dimulai → 403.
- Sewa kendaraan pada tanggal yang sudah terpakai → ditolak saat simpan, kalender menandai harinya.

## 5. Setelah pilot

- [ ] Catat temuan pada tabel di bawah, perbaiki, ulangi alur.
- [ ] Baru lanjut ke Fase 6 (rental, charter, multi-stop, transfer penumpang).

### Catatan temuan

| Tanggal | Peran | Temuan | Tindakan |
|---|---|---|---|
| | | | |
