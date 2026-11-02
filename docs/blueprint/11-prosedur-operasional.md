# Prosedur operasional — GoMad (rilis 1)

Dokumen ini dipakai saat menjalankan GoMad sehari-hari. Setiap prosedur menyebut
**perintah yang benar-benar ada di repo**, bukan langkah manual tanpa jejak.

---

## 0. Pemeriksaan harian (5 menit)

| Waktu | Perintah | Arti hasil |
|---|---|---|
| 02:30 (otomatis) | `php artisan gomad:reconcile` | Setiap akun harus **Cocok: ya** dan jumlah transaksi tidak seimbang = 0. Exit code ≠ 0 berarti ada selisih — eskalasi. |
| tiap 10 menit (otomatis) | `php artisan gomad:money-health` | Exit 1 = masalah kritis (saldo tidak cocok, transaksi tidak seimbang, callback gagal, deposit macet). Semua pesan ditulis ke `Log::critical` sehingga bisa ditangkap alert log. |
| 02:00 (otomatis) | `php artisan app:backup-db` | Backup MariaDB yang disimpan bergilir. |
| tiap 5 menit (otomatis) | `php artisan mad-warung:expire-stale-orders` | Melepas pesanan warung yang kedaluwarsa. |

Jadwal di atas didefinisikan di `routes/console.php` dan dijalankan container
`scheduler`. **Pastikan** `php artisan schedule:list` tidak kosong setelah deploy.

---

## 1. Pencairan dana mitra (withdrawal)

1. Mitra mengajukan dari portal: **Saldo & pencairan**. Minimal Rp 100.000, biaya admin Rp 5.000.
2. Permintaan masuk berstatus `requested`; dana sudah dipindahkan dari
   `BUSINESS.AVAILABLE` ke `PLATFORM.WITHDRAWAL_PAYABLE` (bukan lagi saldo mitra).
3. Admin mengeksekusi transfer bank, lalu menandai `paid` lewat jalur admin.
4. Jika transfer gagal: tandai `failed` → dana dikembalikan ke `BUSINESS.AVAILABLE`
   (kategori `WITHDRAWAL_REVERSAL`). **Jangan** membuat jurnal manual di luar aplikasi.
5. Setelah selesai, jalankan `php artisan gomad:reconcile`. Saldo
   `PLATFORM.WITHDRAWAL_PAYABLE` harus sama dengan jumlah withdrawal `requested`.

**Cek sebelum transfer:** nama pemilik rekening sama dengan nama pemilik usaha, dan
jumlah withdrawal `requested` cocok dengan mutasi bank.

---

## 2. Settlement ke mitra

1. Settlement dilakukan admin (`settlement.execute`) dari `BUSINESS.PAYABLE` → `BUSINESS.AVAILABLE`.
2. Nominal yang disarankan = seluruh `BUSINESS.PAYABLE` mitra pada saat itu.
3. Setelah settlement, mitra dapat langsung mengajukan pencairan.
4. Rincian diskon promo ada di halaman agency (**Diskon & promo**) — promo yang
   ditanggung platform **tidak** mengurangi `BUSINESS.PAYABLE`.

---

## 3. Refund

| Kejadian | Tindakan |
|---|---|
| Pembayaran Midtrans `refund`/`partial_refund` | Webhook memposting jurnal pembalik (`REFUND_APPROVED`) dan mengubah status pembayaran. Tidak ada aksi manual bila berhasil. |
| Refund gagal dibukukan (log: *refund could not be posted*) | Transaksi ditandai perlu rekonsiliasi manual. Jalankan `gomad:reconcile`, periksa mutasi `PLATFORM.CASH`, lalu posting pembalik lewat jalur aplikasi. **Jangan** mengubah `balance_cached` langsung. |
| Sewa dibatalkan setelah lunas | `CancelRental` melepas hold jaminan; deposit yang sudah ditahan dikembalikan lewat `ReturnRentalDeposit`. |
| Deposit dipotong sebagian | Potongan pindah ke `BUSINESS.PAYABLE:<id>`; sisanya kembali ke penyewa. Nominal & alasan tersimpan di baris sewa. |

---

## 4. Pembayaran yang tidak pernah masuk (kegagalan pengiriman / webhook)

1. `gomad:money-health` memberi peringatan untuk `payment_intents` berstatus `pending`
   melewati 4× batas waktu pembayaran.
2. Periksa status sebenarnya di dashboard Midtrans dengan `provider_order_id`.
3. Bila Midtrans menyatakan `settlement` tetapi aplikasi belum: kirim ulang notifikasi dari
   dashboard Midtrans (bukan menulis jurnal manual). Handler webhook idempoten, jadi
   pengiriman berulang aman — dibuktikan tes.
4. Bila Midtrans menyatakan `expire`/`deny`: tidak ada aksi; kursi/booking dilepas otomatis
   oleh scheduler.

---

## 5. Sengketa (dispute) pelanggan

1. Kumpulkan: kode transaksi (`MT-TR-…`/`MT-RN-…`), bukti bayar, dan kronologi.
2. Cek tiga sumber: halaman pesanan pelanggan, portal agency, dan `gomad:reconcile` untuk
   memastikan uang benar-benar sudah tercatat.
3. Hasil sengketa yang mungkin:
   - **Layanan tidak diberikan** → refund penuh (bagian 3).
   - **Sebagian** → refund sebagian atau potongan deposit (bagian 3).
   - **Tidak terbukti** → tutup sengketa dengan catatan; tidak ada mutasi.
4. Setiap sengketa wajib menyebut nomor transaksi di catatan agar bisa diaudit ulang.

---

## 6. Backup & uji pemulihan

**Backup rutin:** `php artisan app:backup-db` (harian 02:00, disimpan bergilir).

**Uji pemulihan (wajib diulang setiap rilis besar):**

Cara tercepat — satu perintah, otomatis:

```bash
cd api
bash bin/verify-backup-restore                      # pakai backup lokal terbaru
bash bin/verify-backup-restore --r2-latest          # unduh backup terbaru dari R2 dulu
bash bin/verify-backup-restore backups/<file>.sql   # berkas tertentu
```

Skrip ini membuat database sekali pakai, memulihkan backup ke sana, lalu **wajib lulus**:
jumlah tabel > 0, `gomad:schema-check` (skema hasil restore = migration), dan
`gomad:reconcile` (saldo ledger cocok). Database sekali pakai dihapus lagi di akhir.

Bila ingin manual:
1. Unduh backup terverifikasi dengan `php artisan app:download-backup`.
2. Pulihkan ke database scratch (bukan produksi).
3. Jalankan `gomad:schema-check` dan `gomad:reconcile`; keduanya harus exit 0.
4. Catat hasilnya di tabel di bawah, lalu hapus database scratch.

| Tanggal | Backup | Hasil | Catatan |
|---|---|---|---|
| 2026-10-30 | `gomad-20261030T130011Z-e86f9ee1.sql` | 49 tabel, skema cocok, rekonsiliasi bersih | Backup dari DB lokal (Aiven tidak dapat dihubungi); verifikasi struktural |

---

## 7. Monitoring & alerting

| Sinyal | Sumber | Ambang | Tindakan |
|---|---|---|---|
| Saldo tidak cocok | `gomad:money-health` (log critical) | ≥ 1 akun | Bekukan pencairan, jalankan `gomad:reconcile --json`, eskalasi |
| Transaksi tidak seimbang | idem | ≥ 1 | idem |
| Callback pembayaran gagal | idem | ≥ 1 | Periksa log `local.ERROR`, ulangi notifikasi Midtrans |
| Deposit masih ditahan padahal sewa selesai | idem | ≥ 1 | Selesaikan lewat `ReturnRentalDeposit` dari portal agency |
| Pembayaran menggantung | idem (peringatan) | > 4× batas waktu | Cek dashboard Midtrans |
| Antrean menumpuk | `queue` container | > 100 job | Periksa worker, restart `queue` |

Alert minimum: notifikasi pada baris `money-health:` dan `critical` di `laravel.log`.

**Kanal alert keluar:** isi `ALERT_WEBHOOK_URL` (Slack/Discord/webhook internal). Setiap kegagalan
kritis dikirim sebagai HTTP POST JSON berisi `severity`, `message`, `context`, `application`,
`environment`, dan `occurred_at`. Bila webhook kosong atau gagal dihubungi, peringatan tetap
masuk ke `laravel.log` — proses uang **tidak** ikut gagal.

---

## 8. Rate limit & keamanan jalur uang

- `throttle:auth` → 5/menit per IP (login, registrasi).
- `throttle:money` → 30/menit per pengguna: ajukan sewa, bayar tiket/sewa.
- `throttle:uploads` → 20/menit: unggah dokumen sewa.
- `throttle:promo` → 10/menit: pembuatan promo.
- Verifikasi dokumen & promo platform hanya untuk `PlatformUserRole role = admin`.
- Berkas dokumen disimpan di disk `local` (privat) dan hanya disajikan lewat stream
  ber-otorisasi; agency kehilangan akses begitu sewa selesai.

---

## 9. Feature freeze

Sejak Fase 8, **tidak ada fitur baru** sampai rilis. Perubahan hanya boleh:
perbaikan bug, penguatan keamanan, atau perbaikan operasional — masing-masing dengan tes.

---

## 10. Uji beban ringan

```bash
cd api
bash bin/load-test            # 200 permintaan, konkurensi 10
bash bin/load-test 300 15     # jumlah permintaan & konkurensi bisa diatur
```

Skrip login sebagai pengguna demo, lalu membebani tiga halaman baca tersibuk
(daftar travel, pencarian dengan tanggal, denah kursi satu jadwal) dan melaporkan
jumlah kegagalan non-2xx, rata-rata, p95, dan maksimum. Exit ≠ 0 bila ada kegagalan.

Hasil terakhir (lokal, satu container app, Docker Compose):

| Tanggal | Beban | Gagal | Rata-rata | p95 | Maks |
|---|---|---|---|---|---|
| 2026-10-30 | 60 permintaan, konkurensi 6 | 0 | 101 ms | 181 ms | 204 ms |
| 2026-10-30 | 300 permintaan, konkurensi 15 | 0 | 209 ms | 267 ms | 311 ms |

Catatan: angka ini untuk lingkungan lokal. Uji pada staging/produksi wajib diulang
setelah domain publik aktif, karena latensi jaringan dan batas platform hosting
berbeda.
