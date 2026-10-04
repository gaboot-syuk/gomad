# Blueprint 07 — Strategi Uji

> **Prinsip:** tes ada untuk menjawab pertanyaan, bukan mengejar angka *coverage*.
> Pertanyaan yang paling penting di proyek ini: **apakah uangnya benar?**
>
> Rujukan: `docs/04-checklist-jangan-diulang` · `blueprint/09` (verifikasi live)

---

## 1. Empat lapis pengujian

| Lapis | Menjawab | Kapan jalan |
|---|---|---|
| **Uji arsitektur** | Apakah batas modul & kesiapan mobile masih dijaga? | Setiap commit |
| **Uji unit domain** | Apakah aturan bisnis benar dalam isolasi? | Setiap commit |
| **Uji integrasi** | Apakah potongan-potongan bekerja bersama? | Setiap commit |
| **Verifikasi live** | Apakah aplikasinya benar-benar bisa dipakai? | **Penutup setiap fase** |

Keempatnya menjawab pertanyaan berbeda. Tidak ada yang bisa menggantikan yang lain.

---

## 2. Lima uji uang — tidak bisa ditawar

Diambil dari `docs/04` bagian F. **Wajib lolos sebelum fase mana pun dinyatakan selesai.**

| # | Uji | Kriteria lulus |
|---|---|---|
| **U1** | Callback duplikat | Kirim callback sama **3×** → saldo berubah **sekali** |
| **U2** | Race condition kursi | Dua permintaan kursi sama, bersamaan → **satu** berhasil |
| **U3** | Jaminan kurang | COD tanpa jaminan cukup → **ditolak di satu titik**, fitur lain tetap jalan |
| **U4** | Rekonsiliasi ledger | Jumlah mutasi **merekonstruksi** saldo setiap akun, di semua skenario |
| **U5** | Rincian settlement | Modal + fee + PPN + hold = net, **tanpa selisih 1 rupiah** |

**Yang membuat U4 sulit:** dia harus benar bukan pada satu skenario, tapi pada **semua kombinasi** — online/cash/COD, penuh/batal/no-show, promo ditanggung platform/business. Karena itu U4 diuji dengan **membangkitkan kombinasi**, bukan menulis satu tes per kasus.

---

## 3. Uji arsitektur

Menegakkan aturan di `blueprint/01` — kalau dilanggar, **CI merah**.

| Aturan | Yang diuji |
|---|---|
| A2 | Core tidak meng-import vertical |
| A3 | Vertical tidak meng-import vertical lain |
| A5 | Tidak ada tabel saldo di luar `Core/Ledger` |
| A6 | Controller tidak memuat logika bisnis |
| **M1** | `Domain/` & `Application/` tidak menyentuh `Request`, `session()`, `auth()`, Inertia |
| **M2** | Action menerima DTO, bukan objek Request |

**Kenapa ini penting:** M1 & M2 terlihat sepele, tapi kalau bocor, **mobile tidak bisa dibangun tanpa menulis ulang logika**. Uji ini yang menjaga janji "web dulu, mobile menyusul" tetap bisa ditepati.

---

## 4. Uji idempotency & webhook

| # | Uji |
|---|---|
| I1 | Callback tanpa signature valid → **ditolak**, tidak ada perubahan data |
| I2 | Callback dengan signature valid tapi duplikat → respons sukses, **tanpa efek ganda** |
| I3 | Callback masuk sebelum transaksi induk ada → **tidak hilang**, diproses setelahnya |
| I4 | `POST` uang tanpa `Idempotency-Key` → ditolak jelas |
| I5 | Callback `failed` dikirim berulang pada pencairan → dana **tidak** dikembalikan berkali-kali |
| I6 | Pelepasan hold dipanggil dua kali → efek **sekali** |

> I5 adalah celah terbesar sistem lama. Dia **wajib** punya tes.

---

## 5. Uji aturan bisnis yang spesifik

Selain lima uji uang, aturan ini perlu tesnya sendiri karena pernah gagal di sistem lama:

| Aturan | Uji |
|---|---|
| `D9` | Hold COD dilepas lewat jalur **tertagih** dan **void** — keduanya bekerja |
| `D6` | Jaminan kurang **tidak** membatalkan tiket yang sudah terjual |
| `P4–P6` | Plafon ditolak di server, bukan hanya di tampilan · approval kedaluwarsa 60 menit |
| `PR2` | Promo platform → `agency_net` **tetap penuh** |
| `PR3` | Promo business → `agency_net` **berkurang** |
| `K5` | Multi-stop: kursi yang turun di titik tengah **bisa dijual lagi** |
| `K3` | Kursi sama pada jadwal sama **ditolak** |
| `T4` | Transfer penumpang memakai penguncian — tidak menyebabkan overbooking |
| `W5` | Penarikan tidak boleh membuat hold > jaminan |
| `RT2` | Deposit penyewa ditagih, ditahan, **dan dikembalikan** |

---

## 6. Uji beban ringan

Bukan untuk membanggakan angka, tapi untuk menemukan hal yang tidak muncul saat sepi.

| Uji | Kriteria |
|---|---|
| 100 pengguna bersamaan mencari jadwal | Waktu balas tetap wajar |
| 20 pemesanan bersamaan pada **satu kursi** | Tepat **1** berhasil, sisanya ditolak jelas |
| 50 callback bersamaan | Tidak ada efek ganda |
| Antrean panjang (notifikasi tertahan) | Sistem tetap melayani permintaan pengguna |

---

## 7. Uji pemulihan

| # | Uji | Kriteria |
|---|---|---|
| B1 | Pulihkan backup ke database kosong | Data utuh, aplikasi jalan |
| B2 | Hitung ulang saldo dari entri ledger | Cocok dengan `balance_cached` |
| B3 | Matikan Redis saat antrean panjang | Tugas tidak hilang, jalan setelah hidup |

> **B1 wajib diuji, bukan sekadar disiapkan.** Backup yang belum pernah dipulihkan belum bisa disebut backup.

---

## 8. Gerbang per fase

Sebuah fase **tidak boleh** dinyatakan selesai kalau salah satu ini belum terpenuhi:

| # | Syarat |
|---|---|
| 1 | Uji otomatis **hijau** — termasuk uji arsitektur |
| 2 | Lima uji uang (**U1–U5**) lolos, kalau fase itu menyentuh uang |
| 3 | **Verifikasi live berhasil** — alur ditelusuri di browser preview (`blueprint/09`) |
| 4 | Keadaan gagal diuji manual: input salah · akses tanpa izin · data kosong |
| 5 | Ditelusuri di **layar kecil** bila fase itu menghasilkan halaman Staff |
| 6 | Tidak ada temuan 🔴 baru di `docs/04-checklist-jangan-diulang.md` |

---

## 9. Anti-pattern yang dihindari

| Anti-pattern | Kenapa dihindari |
|---|---|
| Mengejar angka *coverage* | Coverage tinggi tetap bisa nol tes untuk alur uang |
| Menguji implementasi, bukan perilaku | Tes jadi rusak setiap kali kode dirapikan |
| Menunda tes alur uang "sampai stabil" | Alur uang **tidak akan pernah** stabil kalau tidak diuji |
| Menganggap verifikasi live menggantikan tes otomatis | Live membuktikan bisa dipakai; tes membuktikan tidak bocor |
| Menganggap tes otomatis menggantikan verifikasi live | Tes membuktikan tidak bocor; live membuktikan bisa dipakai |
