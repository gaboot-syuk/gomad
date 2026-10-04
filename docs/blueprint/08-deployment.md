# Blueprint 08 — Deployment & Lingkungan

> Rujukan: `blueprint/09` (Docker & verifikasi live) · `blueprint/06` (Fase 0)
> **Prinsip:** lingkungan bisa dihancurkan dan dibangun ulang kapan saja. Kalau ada langkah manual yang tidak tercatat, itu utang.

---

## 1. Tiga lingkungan

| Lingkungan | Tujuan | Siapa yang pakai |
|---|---|---|
| **Lokal** | Pengembangan & verifikasi live | Pengembang |
| **Staging** | Uji dengan mitra sungguhan — **ber-URL** | Warung · grosir · agency contoh |
| **Produksi** | Rilis lengkap | Publik |

**Perbedaan yang wajib konsisten:** struktur database, daftar parameter, dan versi kode. **Perbedaan yang wajib berbeda:** kunci rahasia, kredensial gateway (sandbox vs produksi), dan data.

**Aturan:** tidak boleh ada perilaku yang hanya bekerja di satu lingkungan, kecuali memang dirancang (mis. gateway sandbox).

---

## 2. CI — setiap commit

| # | Langkah | Gagal → |
|---|---|---|
| 1 | Lint & format | Blokir |
| 2 | **Uji arsitektur** (A1–A7, M1–M2) | Blokir |
| 3 | Uji unit & integrasi | Blokir |
| 4 | **Lima uji uang (U1–U5)** | Blokir |
| 5 | Migrasi bersih dari nol → `migrate:fresh --seed` | Blokir |
| 6 | Bangun aset (*build*) | Blokir |
| 7 | **Generate kontrak API** | Blokir bila berubah tanpa commit |
| 8 | Pemeriksaan kerentanan dependensi | Peringatan |

**Langkah 7 penting karena dua repo:** bila kontrak berubah, repo `mobile/` harus diberi tahu otomatis — bukan menunggu ada yang sadar.

---

## 3. CD

| Lingkungan | Pemicu | Sifat |
|---|---|---|
| Staging | Otomatis dari cabang utama | Boleh tanpa konfirmasi |
| Produksi | **Manual**, dengan konfirmasi | Tidak pernah otomatis |

**Alasannya:** proyek ini menyentuh uang. Rilis otomatis ke produksi berarti kesalahan bisa sampai ke pengguna tanpa satu pun manusia melihatnya.

---

## 4. Rahasia & konfigurasi

| Aturan | Ketentuan |
|---|---|
| R1 | Rahasia **tidak pernah** masuk git — termasuk `.env.example` yang berisi nilai asli |
| R2 | Setiap lingkungan punya kunci sendiri; **jangan** memakai kunci yang sama |
| R3 | Kunci gateway produksi hanya ada di produksi |
| R4 | Rotasi kunci terjadwal, dan **prosedurnya tertulis** |
| R5 | Dokumen ini hanya mencatat **lokasi** rahasia, tidak pernah nilainya |

---

## 5. Backup & pemulihan

| # | Aturan |
|---|---|
| B1 | Database di-backup **harian**; sebelum migrasi besar, backup tambahan wajib |
| B2 | Retensi minimal 30 hari |
| B3 | Backup disimpan **di luar** server aplikasi |
| B4 | **Pemulihan diuji setiap fase besar** — bukan hanya disiapkan |
| B5 | Entri ledger tidak pernah dihapus, jadi ledger sendiri adalah cadangan |

> **B4 adalah syarat lulus Fase 0 dan Fase 8.** Backup yang belum pernah dipulihkan belum bisa disebut backup.

---

## 6. Monitoring & alerting

**Fokusnya bukan "apakah server hidup", tapi "apakah uangnya benar".**

| Yang dipantau | Ambang | Kenapa |
|---|---|---|
| Callback gagal / tidak terverifikasi | > 0 | Indikasi serangan atau kesalahan konfigurasi |
| Callback duplikat tertangkap | Lonjakan | Pengirim mengulang karena tidak menerima balasan |
| Selisih rekonsiliasi harian | **> 0 rupiah** | ⚠️ Prioritas tertinggi |
| Pencairan menggantung | > 24 jam | Uang mitra tertahan |
| Antrean menumpuk | > 500 tugas | Notifikasi & settlement tertunda |
| Kesalahan 5xx | Lonjakan | |
| Stok kursi negatif | > 0 | Seharusnya mustahil — indikasi bug serius |

**Aturan:** setiap peringatan harus punya **tindakan tertulis**. Peringatan yang tidak bisa ditindaklanjuti akan diabaikan, dan setelah diabaikan beberapa kali, semuanya diabaikan.

---

## 7. Prosedur rilis

```
1. Feature freeze — tidak ada fitur baru (Fase 8)
2. CI hijau sepenuhnya
3. Rekonsiliasi penuh: cocok, tanpa selisih
4. Uji pemulihan backup berhasil
5. Uji beban ringan lolos
6. Deployment ke staging · uji asap (smoke test)
7. Mitra contoh menguji alur utama
8. **Verifikasi live di staging** berhasil
9. Backup produksi
10. Deploy produksi (manual)
11. Uji asap produksi
12. Pantau 24 jam pertama
```

**Tidak ada langkah yang boleh dilewati.** Yang paling sering dilewati adalah nomor 3 dan 4 — dan justru keduanya yang melindungi uang.

---

## 8. Rollback

| Keadaan | Tindakan |
|---|---|
| Kode bermasalah, database belum berubah | Kembalikan ke versi sebelumnya |
| Migrasi bermasalah | **Jangan** mengembalikan database secara membabi buta — ledger hanya menerima perbaikan maju |
| Data uang sudah terlanjur berubah | Perbaiki dengan **transaksi pembalik**, bukan `UPDATE` |

| # | Aturan |
|---|---|
| RB1 | Setiap rilis punya versi sebelumnya yang **bisa** dikembalikan |
| RB2 | Migrasi harus punya jalur mundur, atau dijelaskan mengapa tidak mungkin |
| RB3 | **Entri ledger tidak pernah dihapus saat rollback.** Koreksi selalu berupa transaksi baru |

---

## 9. Domain & TLS

| # | Ketentuan |
|---|---|
| D1 | Staging memakai subdomain terpisah, dengan perlindungan kata sandi bila datanya nyata |
| D2 | Produksi wajib HTTPS |
| D3 | Sertifikat diperbarui otomatis, dengan pemantauan kedaluwarsa |
| D4 | Webhook gateway hanya menerima dari daftar alamat yang sah, bila penyedia mendukung |
| D5 | **Staging wajib bisa dibuka mitra** — itu tujuan utamanya ada |

---

## 10. Yang harus sudah ada sebelum Fase 1 dimulai

- [ ] Docker stack jalan (`blueprint/09`)
- [ ] CI menjalankan 8 langkah di bagian 2
- [ ] Staging ber-URL, bisa diakses
- [ ] Backup harian berjalan **dan pemulihannya sudah diuji**
- [ ] Kontrak API digenerate di CI, dan perubahan memberi sinyal ke repo `mobile/`
- [ ] Prosedur rilis ditulis, walau rilis pertama belum terjadi
- [ ] Ambang monitoring terpasang — minimal untuk **selisih rekonsiliasi**
