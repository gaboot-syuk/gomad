# Staging & domain publik — runbook rilis

Tujuan: memindahkan GoMad dari "jalan di laptop" ke **staging ber-URL** yang bisa dipakai
pilot agency sungguhan, lalu produksi di domain publik.

Artefak yang sudah ada di repo:
- `docker/render/Dockerfile` — image produksi (nginx + php-fpm, port **10000**, aset sudah di-`npm run build`).
- `docker/render/nginx.conf`, `docker/render/supervisord.conf`.
- `docker/php/entrypoint.sh` — entrypoint bersama.

---

## 1. Yang perlu disiapkan (prasyarat milik Anda)

| Kebutuhan | Catatan |
|---|---|
| Akun Render (atau VPS) | Web service Docker + **worker** terpisah |
| Database MySQL/MariaDB | Aiven baru **atau** MySQL terkelola; butuh CA untuk TLS |
| Redis | Cache/session/queue (opsional: `database` bila belum ada) |
| Cloudflare R2 | Backup + objek privat (kredensial sudah dipakai lokal) |
| Midtrans | Sandbox untuk staging, produksi untuk rilis |
| Kanal alert | URL webhook Slack/Discord/WhatsApp gateway → `ALERT_WEBHOOK_URL` |

Tanpa akun-akun ini, langkah 2–6 tidak bisa dijalankan oleh siapa pun.

---

## 2. Variabel lingkungan wajib

Nilai rahasia **tidak boleh** masuk repo. Set lewat dashboard hosting.

| Variabel | Contoh / catatan |
|---|---|
| `APP_ENV` / `APP_DEBUG` | `production` / `false` |
| `APP_KEY` | `php artisan key:generate --show` (sekali, simpan di secret) |
| `APP_URL` | `https://staging.gomad.id` atau `https://gomad.id` |
| `APP_TIMEZONE` | **`Asia/Jakarta`** — tanpa ini jam keberangkatan bergeser 7 jam |
| `DB_CONNECTION` | `mysql` |
| `DB_HOST` `DB_PORT` `DB_DATABASE` `DB_USERNAME` `DB_PASSWORD` | kredensial DB terkelola |
| `MYSQL_ATTR_SSL_CA` | path sertifikat CA (mis. `/etc/secrets/aiven-ca.pem`) |
| `CACHE_STORE` `SESSION_DRIVER` `QUEUE_CONNECTION` | `redis` (atau `database`) |
| `MIDTRANS_SERVER_KEY` `MIDTRANS_CLIENT_KEY` `MIDTRANS_MERCHANT_ID` `MIDTRANS_IS_PRODUCTION` | dari dashboard Midtrans |
| `MIDTRANS_SNAP_URL` | `https://app.sandbox.midtrans.com/snap/v1/transactions` (bukan `snap.js`) |
| `MIDTRANS_PAYMENT_TIMEOUT` | `30` (menit) |
| `MIDTRANS_EXPIRE_GRACE_MINUTES` | `15` |
| `MAD_TRANS_SERVICE_FEE` `MAD_TRANS_PLATFORM_FEE_BASIS_POINTS` `MAD_TRANS_PPN_BASIS_POINTS` | `5000` · `300` · **`1100`** |
| `RENTAL_SERVICE_FEE` `RENTAL_PLATFORM_FEE_BASIS_POINTS` `RENTAL_PPN_BASIS_POINTS` | `10000` · `300` · **`1100`** |
| `MAD_WARUNG_*` | fee warung & PPN (lihat `config/services.php`) |
| `REFERRAL_*` `LOYALTY_*` | opsional, ada default |
| `ALERT_WEBHOOK_URL` | **wajib diisi** agar kegagalan uang terkirim keluar |
| `AWS_ACCESS_KEY_ID` `AWS_SECRET_ACCESS_KEY` `AWS_BUCKET` `AWS_ENDPOINT` `AWS_DEFAULT_REGION` `AWS_USE_PATH_STYLE_ENDPOINT` | R2 |

> Catatan: variabel yang dibaca **saat container dibuat** (compose interpolates) berbeda dengan
> variabel runtime biasa. Di Render, semua masuk sebagai env runtime sehingga aman.

---

## 3. Layanan yang harus jalan

| Layanan | Perintah | Alasan |
|---|---|---|
| Web | image `docker/render/Dockerfile`, health check `GET /up` | aplikasi |
| Worker | `php artisan queue:work --tries=3 --timeout=120` | notifikasi & job async |
| Scheduler | `php artisan schedule:work` (atau cron `* * * * * php artisan schedule:run`) | `gomad:money-health` tiap 10 menit, `mad-trans:expire-bookings` tiap 5 menit, `gomad:reconcile` 02:30, backup 02:00, expire warung |

Tanpa worker scheduler, **pelepasan kursi kedaluwarsa dan pemantauan uang tidak berjalan**.

---

## 4. Langkah rilis

```bash
# 1) build & deploy (Render: otomatis dari Dockerfile)
# 2) satu kali: kunci konfigurasi
php artisan key:generate --force      # hanya bila APP_KEY belum ada
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan storage:link

# 3) isi data awal yang WAJIB ada di semua lingkungan
php artisan db:seed --class="App\\Core\\Access\\Infrastructure\\AccessControlSeeder" --force
php artisan db:seed --class="App\\Core\\Ledger\\Infrastructure\\LedgerAccountSeeder" --force

# 4) verifikasi
php artisan gomad:schema-check        # harus exit 0
php artisan gomad:reconcile           # semua akun "Cocok: ya"
php artisan gomad:money-health        # exit 0
```

Staging boleh memakai `php artisan app:seed-demo --confirm` untuk demo.
**Jangan** menjalankan seeder demo di produksi.

Buat akun admin platform pertama (verifikasi dokumen sewa & promo platform):

```bash
# daftarkan dulu akunnya lewat /register, lalu:
php artisan app:assign-platform-admin 0812xxxxxxx
```

---

## 5. Domain & TLS

1. Arahkan DNS `staging.gomad.id` / `gomad.id` ke layanan hosting (Render: otomatis + Let's Encrypt).
2. Paksa HTTPS; `APP_URL` harus memakai `https://`.
3. Pastikan `SESSION_SECURE_COOKIE=true` dan `SANCTUM_STATEFUL_DOMAINS` memuat domain publik.
4. Uji: buka `/up` (harus 200), login, dan satu pemesanan nyata.

---

## 6. Checklist go-live (jalankan berurutan)

- [ ] `/up` mengembalikan 200 dari internet
- [ ] `gomad:schema-check` exit 0
- [ ] `gomad:reconcile` tanpa selisih
- [ ] `gomad:money-health` exit 0 dan **webhook alert terbukti menerima** (uji kirim manual)
- [ ] `bash bin/load-test 300 15` dijalankan ke URL staging → catat p95
- [ ] Scheduler terdaftar: `php artisan schedule:list` tidak kosong
- [ ] Worker memproses job (cek `php artisan queue:monitor`)
- [ ] Backup harian berjalan → `bash bin/verify-backup-restore --r2-latest` berhasil
- [ ] Midtrans: satu transaksi sandbox end-to-end (settle) → saldo ledger benar
- [ ] Login/registrasi + rate limit (`auth` 5/menit) bekerja
- [ ] Dokumen sewa: unggah → admin verifikasi → agency hanya bisa buka saat sewa aktif

---

## 7. Rollback

1. Deploy ulang image sebelumnya (Render: "Rollback" ke deploy terakhir yang sehat).
2. **Jangan** rollback migrasi otomatis. Bila migrasi baru bermasalah, tambahkan migrasi
   perbaikan — `migrate:rollback` hanya untuk kasus yang sudah dianalisis.
3. Setelah rollback: `gomad:schema-check` + `gomad:reconcile` untuk memastikan data konsisten.

---

## 8. Status saat ini (2026-10-30)

| Butir | Status |
|---|---|
| Image produksi (`docker/render`) | ✅ ada |
| Health check `/up` | ✅ aktif |
| Alerting keluar (webhook) | ✅ `SendAlert` + tes (perlu `ALERT_WEBHOOK_URL` diisi) |
| Staging ber-URL | ⬜ **belum** — menunggu akun hosting, DB terkelola, dan DNS |
| Domain publik + TLS | ⬜ **belum** |
| Uji beban lokal | ✅ 300 req / konkurensi 15 → 0 gagal, p95 267 ms |
