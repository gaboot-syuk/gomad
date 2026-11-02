# Blueprint 05 — Aturan Bisnis

> Kumpulan aturan yang mengikat seluruh sistem. **Setiap aturan di sini harus bisa diuji** —
> kalau tidak bisa diuji, desainnya belum selesai.
>
> Rujukan: `docs/02` (deposit gating) · `docs/05` (Core) · `docs/08` (Mad Warung) · `blueprint/02` (skema)

---

## 1. Struktur fee & pajak

### Komponen

| Komponen | Travel | Rental | Mad Warung |
|---|---|---|---|
| Service fee | **Rp 5.000** / tiket | **Rp 10.000** / transaksi | **bulanan** *(angka menyusul)* |
| Platform fee | **3%** × fare | **3%** × harga sewa | **3%** × harga barang |
| PPN | **11%** × (service + platform) | idem | idem |

### Aturan

| # | Aturan |
|---|---|
| F1 | **Penjual menerima harga penuh.** Platform tidak pernah memotong dari harga penjual (Model B) |
| F2 | Buyer membayar: `harga + service fee + platform fee + PPN` |
| F3 | **PPN masuk akun kewajiban**, bukan pendapatan. Kalau tercampur, laba & pelaporan pajak dua-duanya salah |
| F4 | PPN dihitung dari **fee platform**, bukan dari nilai transaksi |
| F5 | Service fee Mad Warung berbasis **bulanan** — supaya frekuensi tinggi tidak dihukum |
| F6 | Semua perhitungan fee memakai **integer rupiah**; pembulatan hanya di tampilan |

### Contoh nyata (Travel, fare Rp 150.000)

```
fare                 Rp 150.000
service fee          Rp   5.000
platform fee (3%)    Rp   4.500   → subtotal komisi Rp 9.500
PPN (11% × 9.500)    Rp   1.045
─────────────────────────────────
dibayar customer     Rp 160.545
diterima agency      Rp 150.000   (utuh)
pendapatan platform  Rp   9.500
```

---

## 2. Deposit gating (COD)

| # | Aturan |
|---|---|
| D1 | Jaminan dihitung **per booking COD**, bukan per kapasitas jadwal |
| D2 | `hold = fee_platform_per_kursi × jumlah kursi COD` |
| D3 | **Gate saat booking COD dibuat:** `COLLATERAL.tersedia ≥ hold` |
| D4 | `COLLATERAL.tersedia = saldo COLLATERAL − total hold aktif` |
| D5 | Hanya berlaku bila jadwal `allow_cod = true` |
| D6 | Jika jaminan kurang → **tolak booking COD baru**. **Jangan** batalkan tiket yang sudah terjual |
| D7 | Jadwal ditandai `risk_status`: `ok` / `warning` / `blocked`; pemilik & ops diberi tahu |
| D8 | Jadwal PP di-hold **per keberangkatan**, bukan per rute |
| D9 | **Pelepasan hold punya dua jalur:** *tertagih* (fee masuk) atau *void* (no-show/batal) |
| D10 | Pelepasan wajib **idempoten** — kunci unik `(source_type, source_id, reason)` |
| D11 | Setelah jadwal selesai & settlement terverifikasi, hold = 0 |
| D12 | COD **no-show dibiarkan** (keputusan produk) — tapi jalur *void* wajib ada, kalau tidak hold menumpuk dan mitra terkunci |

> **Kenapa per booking, bukan per kapasitas:** eksposur platform hanya nyata kalau ada penumpang COD yang benar-benar naik. Mensyaratkan jaminan penuh di muka untuk kursi yang mungkin tidak pernah terjual adalah kesalahan sistem lama (flat Rp 500.000) — cuma dengan rumus lebih rapi.

---

## 3. Plafon penjaga & approval (Mad Warung)

| # | Aturan |
|---|---|
| P1 | Penjaga **punya akun sendiri** dan **dia yang menekan tombol bayar** |
| P2 | Sumber dana = **metode pembayaran milik pemilik**, bukan uang pribadi penjaga |
| P3 | `member_limits` menetapkan: `daily_amount_limit`, `per_order_limit`, `requires_approval_above` |
| P4 | Diperiksa **saat pesanan dibuat** — sebelum pembayaran |
| P5 | Di atas plafon → status `pending_approval` → **approval pemilik dari jauh** |
| P6 | Approval berlaku **60 menit**; lewat itu kedaluwarsa dan harus diminta ulang |
| P7 | Penolakan **wajib disertai alasan** |
| P8 | **Plafon adalah pengaman finansial utama pemilik** — bukan fitur pelengkap. Kalau lemah, proposi "kendali" runtuh |

---

## 4. Shift penjaga

| # | Aturan |
|---|---|
| S1 | Shift untuk **absensi & akuntabilitas**, **bukan** penggerak izin transaksi |
| S2 | Penjaga yang tidak sedang bertugas **tetap bisa** bertransaksi (hanya terekam) |
| S3 | Setiap pesanan mencatat `created_by` + shift yang berlaku saat itu |
| S4 | Laporan pemilik bisa menampilkan aktivitas **per penjaga** dan **per shift** |

---

## 5. Settlement

| Penerima | Pengaturan | Isi yang disetor |
|---|---|---|
| **Agency** | Per agency · default mingguan (Sabtu) | `fare − promo yang ditanggung agency` |
| **Grosir** | Per grosir | `harga barang` (utuh) |

| # | Aturan |
|---|---|
| ST1 | Siklus diatur **per mitra** — bukan satu jadwal seragam |
| ST2 | GoMad **mem-backup pembayaran grosir** di tahap awal → grosir tidak menanggung risiko tidak dibayar |
| ST3 | Backup **tidak butuh modal tambahan** — buyer sudah membayar di muka per pesanan |
| ST4 | Rincian settlement wajib menampilkan: harga penjual · fee platform · PPN · hold · net |
| ST5 | Settlement gagal harus bisa **dibalik** lewat transaksi pembalik |

---

## 6. Pencairan (withdrawal)

| # | Aturan |
|---|---|
| W1 | Minimum **Rp 100.000** |
| W2 | Biaya admin **Rp 5.000** (masuk akun pendapatan) |
| W3 | **Auto-approve ≤ Rp 5.000.000**; di atas itu butuh persetujuan admin |
| W4 | Hanya dari akun `AVAILABLE` |
| W5 | Penarikan **tidak boleh** membuat `total hold aktif > saldo COLLATERAL` |
| W6 | Gagal → **transaksi pembalik**, bukan menimpa entri |
| W7 | Callback pencairan **wajib verifikasi signature + cek status** — ini celah terbesar sistem lama (pengembalian dana berulang) |

---

## 7. Refund & pembatalan

| # | Aturan |
|---|---|
| R1 | **Tidak ada keadaan yang menggantung.** Setiap status refund punya jalur keluar |
| R2 | Jalur: `refund_pending` → `refund_approved` / `refund_rejected` → `refunded` |
| R3 | Sumber dana refund: hak penjual yang belum disetor |
| R4 | Refund tunai lewat warung **tidak ada di Rilis 1** (payment agent tidak dibangun) |
| R5 | Kebijakan pembatalan **wajib tertulis** per moda; default usulan: batal > 24 jam → refund penuh · ≤ 24 jam → potongan 25% · setelah berangkat → tidak ada refund |

---

## 8. Promo & referral

| # | Aturan |
|---|---|
| PR1 | Setiap promo punya **`cost_bearer`**: `platform` atau `business` |
| PR2 | `platform` menanggung → **pendapatan platform berkurang**, `agency_net` **tetap penuh** |
| PR3 | `business` menanggung → `agency_net` berkurang |
| PR4 | **Referral selalu ditanggung platform** |
| PR5 | Kode promo tidak bisa digabung melebihi batas diskon yang ditetapkan |
| PR6 | Rincian diskon **tampil di settlement** penjual |

> **Perbaikan dari sistem lama:** dulu kolom `cost_bearer` ada tapi tidak pernah dibaca — akibatnya promo platform **selalu** memotong agency. Itu bug komersial, bukan sekadar desain.

---

## 9. Kursi, kapasitas & cutoff

| # | Aturan |
|---|---|
| K1 | **Penahanan kursi = durasi pembayaran** (`payment_timeout`, default 30 menit). Tidak ada angka TTL terpisah |
| K2 | Setelah kedaluwarsa, scheduler melepas kursi otomatis |
| K3 | `UNIQUE(schedule_id, seat_number)` untuk booking aktif — **pengaman anti-kursi-dobel** |
| K4 | Kapasitas diambil dari `seat_layout` kendaraan — **bukan** akumulasi jumlah penumpang |
| K5 | **Kapasitas dihitung per segmen.** Kursi yang ditinggalkan di titik tengah **bisa dijual lagi** |
| K6 | Kursi supir tidak bisa dipesan |
| K7 | **Cutoff penjualan:** travel 60 menit · rute panjang / door-to-door banyak titik 120 menit — bisa diatur per rute |
| K8 | Semua jalur (booking, transfer, input manual) memakai **logika kapasitas yang sama** — tidak boleh ada duplikasi |
| K9 | **Denah kursi dibentuk dari formasi baris**, bukan diketik per kode. Owner memilih jumlah kursi tiap baris (mis. `2-3-3` untuk Hiace) dan sistem membentuk `A1,A2,B1,B2,B3,C1,C2,C3` (`App\Core\MadTrans\Domain\SeatLayout`) |
| K10 | Batas formasi: maksimal **8 baris**, **4 kursi per baris**, **40 kursi** per kendaraan. Baris 0 dilewati tetapi huruf barisnya tetap (A, C, D — B kosong tidak dipakai ulang) |
| K11 | **Kursi supir ditandai terpisah** lewat `vehicles.has_driver_seat`, bukan sebagai kursi di `seat_layout`. Ia tidak pernah masuk kapasitas jual dan tidak pernah muncul sebagai kursi yang bisa dipilih |
| K12 | `seat_layout` tetap satu-satunya sumber kebenaran penjualan: formasi hanya **cara mengisinya**, bukan tabel kedua |

---

## 9a. Biaya jemput door to door

Titik kumpul tetap titik berat perjalanan. Pelanggan **boleh** meminta dijemput di
luar titik kumpul pada kota yang sama; sampai radius tertentu gratis, selebihnya
dihitung per kilometer. Kebijakan ini milik usaha (`business_pickup_policies`),
jadi tidak perlu migrasi saat agency mengubah tarif.

| # | Aturan |
|---|---|
| J1 | Kebijakan per usaha: `free_radius_km` (default 0) dan `fee_per_km`. Satu baris per usaha |
| J2 | Jarak dihitung **haversine** dari koordinat titik kumpul ke koordinat jemput, dibulatkan 2 desimal |
| J3 | Kilometer yang ditagih = `ceil(jarak − free_radius_km)`. **Pembulatan ke atas**: 11,1 km dengan radius gratis 5 km → 7 km yang ditagih |
| J4 | Biaya = `km_ditagih × fee_per_km`, dalam rupiah bulat. Radius 0 berarti ditagih sejak kilometer pertama |
| J5 | Tanpa kebijakan, tanpa koordinat titik kumpul, atau tanpa titik jemput → **biaya 0**, bukan galat. Door to door bersifat opsional |
| J6 | **Biaya jemput masuk ke pendapatan usaha** (ongkos angkut), **tidak** masuk basis platform fee maupun PPN. Rumus fee & pajak tidak berubah karena ada jemput |
| J7 | Saat booking dibatalkan, biaya jemput mengikuti porsi pengembalian di §7 — bukan dikecualikan |
| J8 | Titik jemput disimpan pada booking (`pickup_address`, `pickup_latitude`, `pickup_longitude`, `pickup_distance_km`, `pickup_fee`) supaya angka saat memesan bisa diaudit |
| J9 | Server menghitung, klien hanya menampilkan. Halaman pilih kursi memanggil `pickup-quote` setiap titik berubah; tidak ada perhitungan biaya di browser |
| J10 | Koordinat titik kumpul disimpan di `route_stops.latitude/longitude`. Owner mengisinya dengan menempel tautan Google Maps/OpenStreetMap atau koordinat `lat, lng` (`GeoPoint::fromInput`) |
| J11 | Peta memakai **tautan OpenStreetMap**, bukan SDK berbayar — tidak ada biaya peta dan pelanggan memakai aplikasi peta yang sudah ia punya |

---

## 9b. Kasir warung (POS)

Penjualan langsung ke pembeli yang datang ke warung. **Bukan** bagian dari
`orders` (pesanan warung ke grosir) dan **bukan** transaksi platform.

| # | Aturan |
|---|---|
| P1 | Harga jual diambil dari `retail_listings` di server. Harga yang dikirim klien **diabaikan** — kalau tidak, siapa pun bisa menjual seharga Rp 0 |
| P2 | Barang masuk daftar jual saat warung **menerima** pesanan grosir, dengan harga beli terakhir sebagai titik awal. Harga yang sudah disesuaikan pemilik tidak pernah ditimpa |
| P3 | **Tanpa biaya platform dan tanpa PPN.** `total_amount` = jumlah baris. Platform bukan penjual dan tidak memproses pembayarannya |
| P4 | **Tidak ada posting ledger.** Kas `BUSINESS.AVAILABLE:<id>` berarti uang yang dipegang platform; tunai kasir tidak pernah sampai ke sana. Nota kasir tetap tersimpan dan bisa diaudit |
| P5 | **Stok grosir tidak dikurangi.** `product_prices.stock_available` milik grosir, bukan rak warung. Persediaan warung belum dimodelkan |
| P6 | Uang diterima kurang dari total → transaksi ditolak. Kembalian dihitung server, bukan hanya ditampilkan |
| P7 | Baris nota menyimpan **salinan** nama, satuan, dan harga saat transaksi; perubahan harga besok tidak boleh mengubah nota kemarin |
| P8 | Nota mencatat shift aktif penjaga bila ada, sehingga penjualan bisa dikelompokkan per shift |
| P9 | Izin: `pos.sell` untuk penjaga, `pos.report` untuk pemilik. Penolakan dilakukan di action, bukan hanya dengan menyembunyikan menu |

---

## 10. Transfer penumpang

| # | Aturan |
|---|---|
| T1 | Biaya = `transfer_fee_per_passenger` (default **Rp 20.000**), dibatasi `max_transfer_fee_percent` (**20%** dari harga tiket) |
| T2 | Butuh persetujuan agency |
| T3 | Harus memindahkan **kursi** dan **uang** dengan benar di **kedua** jadwal |
| T4 | Harus memakai penguncian yang sama seperti booking biasa — di sistem lama jalur ini **tanpa lock**, penyebab overbooking |
| T5 | Jika jadwal tujuan penuh → permintaan ditolak dengan jelas, bukan gagal separuh jalan |

---

## 11. Rental: OTS & deposit penyewa

| # | Aturan |
|---|---|
| RT1 | **OTS** (bayar di tempat) punya risiko seperti COD → berlaku **gate jaminan yang sama** |
| RT2 | **Deposit penyewa wajib ditagih, ditahan, dan dikembalikan.** Di sistem lama kolomnya ada tapi tidak pernah berfungsi |
| RT3 | Deposit dikembalikan setelah kendaraan kembali **dan** inspeksi selesai |
| RT4 | Lepas kunci wajib **verifikasi KTP + SIM + selfie** |
| RT5 | Kendaraan tidak bisa dobel-booking — kalender ketersediaan mengunci rentang tanggal |
| RT6 | Pengurangan deposit (kerusakan/denda) **wajib disertai alasan & bukti** |

---

## 12. Notifikasi — pemicu wajib

| Penerima | Pemicu |
|---|---|
| Pemilik | Pesanan penjaga di atas plafon (butuh approval) · saldo jaminan menipis · penjaga mulai bertugas |
| Penjaga | Approval disetujui/ditolak · pesanan siap diambil · pengingat saldo jaminan |
| Grosir | Pesanan baru masuk · pesanan akan disetor |
| Agency | Jadwal baru diajukan supir · pembayaran diterima · settlement disetor |
| Customer | Pembayaran berhasil/gagal · tiket terbit · pengingat keberangkatan · pembatalan/refund |

| # | Aturan |
|---|---|
| N1 | Notifikasi **push-ready sejak awal** — jangan menambah push belakangan |
| N2 | Kegagalan kirim **dicatat & dicoba ulang**, tidak hilang diam-diam |
| N3 | Notifikasi uang (approval, settlement) **tidak boleh** bisa dimatikan pengguna |

---

## 13. Aturan yang wajib diuji

Lima ini tidak boleh lolos ke produksi tanpa tes otomatis (rincian di `07-strategi-uji.md`):

1. **Callback duplikat** dikirim 3× → saldo berubah **sekali**
2. **Dua permintaan kursi sama** bersamaan → **satu** berhasil
3. **Jaminan kurang** → COD ditolak **di satu titik**, tanpa memblokir fitur lain
4. **Rekonsiliasi** — jumlah mutasi merekonstruksi saldo setiap akun
5. **Rincian settlement** — tanpa selisih 1 rupiah

---

## 14. Tabel parameter

**Setiap parameter wajib punya pemakai. Kalau tidak, jangan ditulis.** (Pelajaran: di sistem lama `schedule_min_days_before = 30` tidak pernah dipakai.)

| Parameter | Default | Dapat diubah | Pemakai |
|---|---|---|---|
| `payment_timeout` | 30 menit | admin | `K1` |
| `service_fee.travel` | Rp 5.000 | admin | `F1` |
| `service_fee.rental` | Rp 10.000 | admin | `F1` |
| `service_fee.mad_warung` | **Rp 0** di Rilis 1 | admin | `F5` — lihat bagian 16 |
| `plafon.per_order` | **1,5× rata-rata kulakan** | pemilik | `P3` — lihat bagian 16 |
| `plafon.daily` | **4× rata-rata kulakan** | pemilik | `P3` — lihat bagian 16 |
| `plafon.fallback_per_order` | Rp 500.000 | admin | `P3` — bila pemilik tidak tahu |
| `plafon.fallback_daily` | Rp 1.500.000 | admin | `P3` |
| `platform_fee_rate` | 3% | admin | `F1` |
| `ppn_rate` | 11% | admin | `F4` |
| `cutoff.travel_minutes` | 60 | per rute | `K7` |
| `cutoff.long_route_minutes` | 120 | per rute | `K7` |
| `transfer.fee_per_passenger` | Rp 20.000 | per jadwal | `T1` |
| `transfer.max_fee_percent` | 20% | per jadwal | `T1` |
| `withdrawal.min_amount` | Rp 100.000 | admin | `W1` |
| `withdrawal.admin_fee` | Rp 5.000 | admin | `W2` |
| `withdrawal.auto_approve_limit` | Rp 5.000.000 | admin | `W3` |
| `approval.expiry_minutes` | 60 | admin | `P6` |
| `settlement.agency_default` | mingguan · Sabtu | per agency | `ST1` |
| `settlement.grosir_default` | **real-time saat penjaga konfirmasi terima** | per grosir | `ST1` — lihat bagian 16 |
| `cancellation.tiers` | >24 jam penuh · ≤24 jam potong 25% | admin | `R5` |

---

## 15. Status keputusan

### Sudah diputuskan

| # | Item | Keputusan |
|---|---|---|
| 1 | **Asumsi PPN** | ✅ **Ditambahkan ke pembeli** → `F2`, `F4` berlaku apa adanya |
| 2 | **Pemelihara katalog grosir** | ✅ **Hibrida** — GoMad isi awal, grosir update harga & stok |
| 3 | **Fallback pembayaran** | ✅ Metode default (token) + **fallback QRIS/VA** |
| 4 | **Kebijakan pembatalan** (`R5`) | ✅ >24 jam refund penuh · ≤24 jam potong 25% · setelah berangkat tidak ada |
| 5 | **Service fee Mad Warung** | ✅ **Rp 0 di Rilis 1** — lihat bagian 16 |
| 6 | **Plafon penjaga** | ✅ Diturunkan dari volume warung, + fallback — lihat bagian 16 |
| 7 | **Settlement grosir** | ✅ Dipicu **bukti terima**, bukan siklus — lihat bagian 16 |
| 8 | **Prefix kode referensi** | ✅ **Dipisah per vertical** (`MW-`, `MT-`, `WD-`, `ST-`) |
| 9 | **Wilayah** | ✅ Data referensi **seluruh Indonesia** sejak awal; fokus rilis **Jabodetabek** |

### Di luar Rilis 1 — dikonfirmasi

Asuransi perjalanan · monitoring kendaraan (IoT) · corporate & event · premium agency · **warung premium** · iklan · **skala Mad Warung dari B2B ke B2B2C** · analitik

---

## 16. Rekomendasi parameter & alasannya

### 16.1 Service fee Mad Warung — **Rp 0 di Rilis 1**

**Rekomendasi: hapus service fee untuk Mad Warung di Rilis 1.** Hanya platform fee 3% + PPN.

| Alasan | Penjelasan |
|---|---|
| **Langganan butuh penagihan** | Fee persentase tertarik otomatis tiap transaksi. Langganan bulanan butuh penagihan, pengingat, dan pemutusan — **beban operasional baru** |
| **Ruangnya cuma 5%** | Celah harga agen hanya 5%. Tidak cukup untuk menanggung biaya penagihan langganan |
| **Frekuensi tidak boleh dihukum** | Warung kulakan 2× sehari. Fee tetap apa pun basisnya menambah friksi |
| **Nilai jual paling kuat** | Dengan 3% + PPN saja, warung **selalu hemat 1,6%** di semua ukuran pesanan |
| **Sudah ada jalurnya nanti** | "Warung premium" sudah dikonfirmasi **di luar Rilis 1** — pendapatan berulang bisa menyusul di sana |

**Kalau kamu tetap ingin ada service fee bulanan:** pakai **Rp 49.000/bulan** (di bawah batas psikologis 50rb), dan **tarik otomatis pada pesanan pertama setiap bulan** — bukan lewat penagihan terpisah. Dengan begitu tidak ada pekerjaan penagihan, dan tidak ada pelanggan yang merasa ditagih.

### 16.2 Plafon penjaga — **diturunkan dari volume warung**

**Rekomendasi: jangan tetapkan angka nasional. Turunkan dari angka pemilik sendiri.**

Saat onboarding, pemilik ditanya satu hal: *"biasanya sekali kulakan berapa?"* → misal Rp 500.000. Sistem menetapkan:

```
per_order_limit           = 1,5 × angka pemilik   → Rp  750.000
requires_approval_above   = sama dengan per_order_limit
daily_amount_limit        = 4 × angka pemilik     → Rp2.000.000
```

**Kalau pemilik tidak tahu — fallback:** per pesanan **Rp 500.000** · harian **Rp 1.500.000**.

| Alasan |
|---|
| **Angka absolut salah untuk setengah pengguna.** Warung di area ramai bisa 3× volume warung di gang sepi |
| **Default terlalu tinggi merusak kepercayaan.** Pemilik yang menemukan penjaga bisa menghabiskan terlalu banyak akan kehilangan kepercayaan pada platform |
| **Default terlalu rendah membebani penjaga.** Penjaga jadi sering minta approval untuk hal wajar |
| **Mulai konservatif, naikkan berdasarkan data** — pemilik yang menaikkan lebih baik daripada pemilik yang merasa dirugikan |

**Tambahan yang saya sarankan:** pemilik harus bisa melihat **berapa kali plafon ditembus**. Itu sinyal plafonnya salah setel — kalau sering, naikkan; kalau tidak pernah, bisa diturunkan. Tanpa sinyal ini, plafon jadi angka yang dipasang sekali lalu dilupakan.

### 16.3 Settlement grosir — **dipicu bukti terima, bukan siklus**

**Rekomendasi: dana masuk saldo grosir begitu penjaga menekan konfirmasi penerimaan.** Bukan harian, bukan mingguan.

```
Penjaga konfirmasi terima  →  saldo grosir bertambah  →  grosir cairkan kapan saja
```

| Alasan | Penjelasan |
|---|---|
| **Gratis** | Menambah saldo itu hanya pencatatan. Biaya bank baru muncul saat **pencairan ke rekening** |
| **Terikat pengaman** | Bukti terima sudah kita wajibkan karena GoMad membayar lebih dulu. Jadi dana bergerak tepat saat risikonya sudah tertutup |
| **Sinyal kepercayaan terkuat** | Grosir paling takut melepas barang lalu tidak dibayar. "Dana masuk begitu barang diterima" menghapus kekhawatiran itu |
| **Tidak ada pekerjaan berkala** | Tidak ada proses settlement terjadwal yang bisa gagal atau tertunda |
| **Grosir tetap bisa memilih** | Yang butuh pola mingguan tetap bisa — hanya pencairannya yang diatur, bukan penerimaan dana |

**Pencairan ke rekening:** kapan saja, minimum **Rp 100.000**, biaya admin **Rp 5.000** — sama seperti agency.

> **Agency tetap memakai siklus** (default mingguan Sabtu). Perbedaannya masuk akal: agency menerima uang dari penumpang yang sudah membayar di muka lewat platform, sedangkan grosir menanggung risiko barang lebih dulu.
