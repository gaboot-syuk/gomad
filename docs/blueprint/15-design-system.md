# Blueprint 15 — Design System di Kode

> **Fondasi tahap D0, diperbarui ke Unified Design System v1.0.** Satu identitas
> merek, **dua bahasa desain**, dan satu tempat untuk mengubahnya. Rujukan nilai:
> `system-design-new.md` di akar repo (UDS) — dokumen itu yang memutuskan warna,
> skala, tema, dan nada; berkas ini hanya mencatat **di mana** keputusan itu hidup
> di repo ini.
>
> Rujukan: `blueprint/04` (layar per peran) · `blueprint/14` (peta web → widget)

---

## 1. Prinsip

| # | Prinsip | Konsekuensi |
|---|---|---|
| 1 | **Token, bukan hex di markup** | Semua warna/spasi/radius lewat `resources/css/app.css` (UDS §13.4) |
| 2 | **Skala Tailwind dibersihkan** | Palet bawaan dibuang supaya tidak ada dua sumber warna |
| 3 | **Satu fondasi, dua tema** | Operasional = Classic Minimalist; Consumer = Glassmorphism (UDS §3.2) |
| 4 | **Status sistem ≠ warna merek** | `success/warning/error/info` terpisah dari `gomad-lime`/`gomad-green` (UDS §4.3) |
| 5 | **Kaca hanya di consumer** | Tema operasional tidak boleh memakai blur (UDS §16.3) |
| 6 | **Empat keadaan halaman** | memuat · kosong · galat · terisi — kosong & galat dirancang |
| 7 | **Satu komponen, satu arti** | Kartu, badge, tombol, dan nominal tidak direka per halaman |

---

## 2. Token (UDS §4–§8)

| Kelompok | Nilai | Dipakai untuk |
|---|---|---|
| Merek utama | `gomad-lime #a8e62e` (Electric Lime), `gomad-lime-fresh #dfff52` | Aksen merek, CTA konsumen |
| Merek sekunder | `gomad-green #258447` (Vibrant Green), `gomad-green-deep #174d38` (Deep Green) | Permukaan kuat, tombol utama, teks merek |
| Aksen | `gomad-yellow #f6d83d` (Sunny Yellow) | Sorotan, sorotan ekonomi |
| Skala lime/hijau/kuning | 50–900 | Hover, aktif, latar lembut |
| Netral | `gray-50 #f7faf2` (Cloud White) · `gray-100 #f1f3ef` (Soft Gray) · `gray-200 #e4e8e1` (Border) · `gray-500/600` (teks sekunder) · `black-900 #263b2b` (Deep Olive) | Teks, garis, latar |
| Permukaan | `surface-base`, `surface-subtle`, `surface-raised` | Latar aplikasi, bidang sekunder, kartu |
| Status | `success #258447`, `warning #d99a18`, `error #d64545`, `info #3278b7` | Badge & pesan |
| Radius | `sm 6 · md 10 · lg 14 · xl 20 · 2xl 28 · pill` | Kartu, input, tombol, kaca |
| Bayangan | `sm · md · lg · xl · glass` | Hemat di operasional; `glass` hanya di consumer |
| Kaca | `gm-glass` (65% + blur 16), `gm-glass-strong` (82% + blur 24) | Navigasi mengambang, panel ringkasan |
| Tipografi | **Inter** 400–800 (UDS §5.1) | Seluruh produk; Montserrat hanya untuk kampanye |

**Wadah:** utilitas `gm-container` (maks **1440px**; padding 16 → 24 → 32 → 40px sesuai
UDS §6.2) dan `gm-container-narrow` (760px) untuk halaman baca panjang.

**Dua penyimpangan yang disengaja, dengan alasan:**

1. `gray-500/600` memakai `#6b776c`/`#5e6a5f`, bukan Muted Gray `#7d887d` dari UDS §4.2.
   Nilai dokumen itu hanya mencapai kontras 3,69:1 terhadap putih sehingga gagal
   UDS §12.1 (4,5:1). Nilai asli tetap tersedia sebagai `gomad-muted` untuk permukaan.
2. Judul halaman operasional memakai 30px (H2), bukan H1 36px; kepadatan informasi
   operasional lebih penting daripada judul besar (UDS §7.4).
3. **Garis tepi kaca memakai hairline gelap** `rgb(38 59 43 / 0.10)`, bukan putih
   `rgba(255,255,255,0.75)` dari UDS §8.3. Di atas latar gradien terang, tepi putih
   membuat kartu kehilangan batas dan terbaca sebagai bidang yang bocor. UDS §8.3
   sendiri menyatakan nilainya adalah "initial design specifications and must be
   validated during implementation".
4. **Formulir kompleks tidak memakai kaca.** UDS §8.4 meminta kaca dihindari pada
   *complex forms*; karena itu pendaftaran (`shared/auth/register`) memakai permukaan
   padat, sedangkan masuk (`login`, dua kolom) tetap kaca.

---

## 2a. Dua tema dan cara memilihnya

| Tema | `data-theme` | Kerangka | Dipakai oleh |
|---|---|---|---|
| Classic Minimalist | `operational` | `layouts/partials/operational-shell` — sidebar, top bar, permukaan padat | Admin, Ops, Grosir, Retail, Penjaga warung, Owner/staf agency, Supir |
| Minimalist Glassmorphism | `consumer` | `layouts/partials/consumer-shell` — navigasi mengambang, navigasi bawah | Pelanggan, seluruh halaman publik |

Tema **tidak** dipilih per halaman. `App\Support\DesignTheme::for($user, $routeName)`
menerapkan urutan UDS §16.2:

```
IF rute publik (home, login, register, mfa.*, invitations.*, public.*, warung.public.*)
    → consumer            (UDS "product == Public")
ELSE IF peran == Customer
    → consumer
ELSE
    → operational
```

Konsekuensi yang dijaga: pemilik grosir yang membuka `/` melihat bahasa desain publik
(bukan sidebar), dan tetap mendapat tombol “Ruang kerja” untuk kembali. Tema **bukan**
mekanisme keamanan (UDS §14.3) — middleware `area:` tetap satu-satunya penjaga.

Komponen membaca tema lewat `DesignTheme::current()`, sehingga `x-gm.card` otomatis
kaca di consumer dan padat di operasional tanpa setiap view mengirim flag.


---

## 3. Komponen bersama

| Komponen | Berkas | Catatan |
|---|---|---|
| Kartu | `components/gm/card.blade.php` | Header/isi/aksi/footer; kaca di consumer, padat di operasional |
| Tombol | `components/gm/button.blade.php` | `primary` (Deep Green), `brand` (Lime), `secondary`, `ghost`, `danger` |
| Badge | `components/gm/badge.blade.php` | `neutral`, `brand`, `success`, `warning`, `error`, `info` |
| Nominal | `components/gm/money.blade.php` + `App\Support\Rupiah` | `Rp 150.000`; aturan di kelas, bukan di view |
| Pesan | `components/gm/alert.blade.php` | `role=alert` untuk galat, `role=status` untuk info |
| Keadaan kosong | `components/gm/empty.blade.php` | Selalu punya langkah berikutnya |
| KPI | `components/gm/stat.blade.php` | Dasbor |
| Header halaman | `components/gm/page-header.blade.php` | Eyebrow + judul + subjudul + aksi |

**Utilitas tema di `app.css`:** `gm-glass`, `gm-glass-strong`, `gm-surface`,
`gm-sticky-cta`, `gm-scroll-x` (geser mendatar tanpa batang gulir), `gm-container`.

**Aksesibilitas yang sudah dikunci di CSS:** fokus `2px` hijau dengan offset, target
sentuh ≥ 44px pada navigasi bawah, `prefers-reduced-motion`, dan **fallback kaca padat**
lewat `@supports not (backdrop-filter: …)` sehingga halaman tetap terbaca di perangkat
atau browser tanpa blur (UDS §12.10).

---

## 4. Navigasi sadar peran

`App\Support\Navigation` menyusun menu dari **izin yang sama** yang menjaga rutenya
(`ActionAuthorizer`), bukan dari daftar halaman. Menu dan penjagaan karena itu tidak
bisa berbeda pendapat.

| Peran | Menu |
|---|---|
| Pengunjung | Cari jadwal · Mad Warung · Jadi mitra · Unduh aplikasi |
| Pengguna | Beranda · Cari jadwal · Pesanan saya · Mad Warung · Level & misi · Referral |
| Pemilik/staf usaha | ditambah Portal agency (bila punya agency) atau Outlet saya (bila punya warung/grosir) |
| Admin platform | ditambah Admin (bila punya izin `withdrawal.review` atau `settlement.execute`) |

---

## 5. Lapisan transisi

Banyak halaman lama memakai markup sederhana (`.card`, `.row`, `.muted`) dan kontrol
formulir polos. Supaya penggantian layout tidak merusak halaman, `app.css` menyediakan:

- kelas `.card/.row/.muted/.success/.error/.inline` di `@layer components`;
- gaya dasar `input/select/textarea/label/button/table` di `@layer base` memakai `:where()`
  supaya spesifisitasnya nol dan **selalu kalah** dari kelas komponen.

Lapisan ini boleh dihapus setelah semua halaman memakai komponen `gm.*`.

---

## 6. Status

| Tahap | Status |
|---|---|
| D0 — token, layout, komponen, navigasi | ✅ selesai |
| D1 — permukaan publik | ✅ selesai (tema consumer) |
| D2 — customer | ✅ selesai (tema consumer) |
| D3 — agency | ✅ selesai |
| D4 — warung/grosir | ✅ selesai |
| D5 — admin/ops | ✅ selesai |
| D6 — bersama (dukungan + pelacakan + MFA) | ✅ selesai |
| **D7 — Unified Design System v1.0** | 🟡 fondasi selesai; halaman menyusul |

### D7 — apa yang sudah dan belum

| Bagian | Status |
|---|---|
| Token UDS §4/§5/§6/§8 di `app.css` | ✅ palet lama dihapus, Inter, skala 1440px |
| Dua kerangka tema (`operational-shell`, `consumer-shell`) | ✅ |
| `App\Support\DesignTheme` + `AccountType::designLanguage()` | ✅ |
| Komponen `gm.*` sadar-tema | ✅ kartu, tombol, badge, KPI, keadaan kosong, header |
| Beranda publik | ✅ hero kaca + blok ajakan mitra Deep Green |
| Halaman pilih kursi | ✅ denah formasi + kursi supir + titik jemput door to door |
| Halaman lain (search, warung, akun, admin, agency, driver) | 🟡 mengikuti tema otomatis, komposisi belum ditata ulang |

**Pola yang wajib ditiru tahap berikutnya:** satu halaman per pekerjaan (bukan satu
halaman raksasa), kepala halaman `x-gm.page-header`, tab pekerjaan (`madtrans/travel/_nav`,
`madwarung/_nav`, `platform/admin/_nav`), dan setiap halaman punya endpoint API pasangannya.
Saldo usaha dibaca dari `App\Core\Ledger\Application\BusinessWalletSummary` supaya web dan
API tidak pernah menampilkan angka berbeda.
