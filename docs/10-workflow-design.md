# Workflow Design

## 1. Tujuan Dokumen

Dokumen ini mendefinisikan alur proses utama pada Sistem Pendataan dan Rekapitulasi Zakat Fitrah Desa.

Workflow Design digunakan untuk menjelaskan:

* bagaimana data bergerak di dalam sistem;
* bagaimana pengguna berinteraksi dengan proses;
* bagaimana sistem melakukan validasi;
* bagaimana pencatatan keluarga dan pembayaran berhubungan;
* bagaimana status rekap lokasi berubah;
* kapan suatu data dapat diubah atau dikunci;
* bagaimana proses revisi dilakukan;
* bagaimana rekap lokasi menjadi bagian dari rekap resmi desa.

Dokumen ini menjadi dasar untuk:

* System Design;
* Architecture Design;
* Database Design;
* API Design;
* Authorization Design;
* UI/UX Flow;
* Testing.

---

# 2. Konsep Utama Workflow

Sistem menggunakan tiga konsep data utama:

```text
KELUARGA / PENCATATAN KELUARGA
              │
              │ memiliki
              ▼
         PEMBAYARAN
              │
              │ dihitung ke
              ▼
        REKAP LOKASI
              │
              │ jika APPROVED
              ▼
         REKAP DESA
```

## 2.1 Pencatatan Keluarga

Pencatatan keluarga merepresentasikan keluarga yang melakukan pembayaran zakat pada suatu periode dan konteks lokasi.

Satu keluarga dapat memiliki lebih dari satu pembayaran.

Contoh:

```text
Keluarga Ahmad
│
├── Pembayaran #1
│   ├── 2 jiwa
│   ├── 5 kg beras
│   └── 10 Maret
│
└── Pembayaran #2
    ├── 1 jiwa
    ├── pembayaran uang
    └── 11 Maret
```

---

## 2.2 Pembayaran

Pembayaran merupakan kejadian pembayaran zakat yang dicatat oleh Petugas Lokasi.

Satu pembayaran dapat berupa:

* beras;
* uang;
* kombinasi beras dan uang.

Pembayaran baru tidak boleh menimpa pembayaran sebelumnya.

---

## 2.3 Rekap Lokasi

Rekap Lokasi merupakan agregasi seluruh pembayaran valid pada:

* satu lokasi;
* satu periode zakat.

Rekap lokasi memiliki workflow status tersendiri.

Status konseptual V1:

* `DRAFT`
* `SUBMITTED`
* `REVISION_REQUIRED`
* `APPROVED`

---

# 3. Workflow 1 — Pendataan Pembayaran Zakat

## 3.1 Tujuan

Mencatat pembayaran zakat fitrah yang dilakukan warga melalui Petugas Lokasi.

## 3.2 Actor

Petugas Lokasi.

## 3.3 Preconditions

* Petugas telah login.
* Akun Petugas aktif.
* Petugas memiliki lokasi penugasan.
* Lokasi masih aktif.
* Periode zakat sedang aktif.
* Rekap lokasi berada pada status yang memperbolehkan perubahan data.

## 3.4 Main Workflow

```text
Warga datang
      │
      ▼
Petugas memilih
Tambah Pembayaran
      │
      ▼
Sistem menampilkan form
      │
      ▼
Petugas mengisi:
- Kepala keluarga
- Informasi identifikasi keluarga
- Jumlah jiwa
- Jiwa beras
- Jumlah beras
- Jiwa uang
- Nominal uang
- Tanggal pembayaran
      │
      ▼
Petugas memilih SIMPAN
      │
      ▼
Sistem validasi
      │
      ├── Tidak valid
      │       ↓
      │   Tampilkan error
      │       ↓
      │   Petugas memperbaiki
      │
      └── Valid
              ↓
      Deteksi keluarga
              ↓
      Proses Workflow 2
```

## 3.5 Validation Rules

Sebelum melanjutkan proses, sistem harus memastikan:

* jumlah jiwa minimal 1;
* minimal terdapat pembayaran beras atau uang;
* jumlah beras tidak negatif;
* nominal uang tidak negatif;
* jumlah jiwa beras dan jiwa uang konsisten;
* lokasi tersedia;
* periode tersedia dan aktif;
* tanggal pembayaran valid.

Apabila menggunakan pembayaran campuran:

```text
Jiwa Beras + Jiwa Uang = Total Jiwa Pembayaran
```

Contoh:

```text
Total jiwa : 5
Jiwa beras : 3
Jiwa uang  : 2

3 + 2 = 5 → VALID
```

---

# 4. Workflow 2 — Deteksi Keluarga

## 4.1 Tujuan

Mencegah pencatatan keluarga ganda sekaligus mempermudah Petugas tanpa mengharuskannya melakukan pencarian manual terlebih dahulu.

## 4.2 Prinsip

Petugas mengisi formulir terlebih dahulu.

Pencarian terhadap keluarga yang sudah tersedia dilakukan oleh sistem setelah Petugas memilih Simpan.

Sistem bertugas mencari.

Petugas bertugas mengonfirmasi apabila terdapat kandidat yang berpotensi sama.

## 4.3 Workflow

```text
Form pembayaran valid
        │
        ▼
Sistem mencari keluarga
pada periode/lokasi terkait
        │
        ▼
Apakah terdapat kandidat?
        │
    ┌───┴────┐
    │        │
  TIDAK      YA
    │        │
    ▼        ▼
Buat        Tampilkan
keluarga    kandidat
baru        keluarga
    │        │
    │        ▼
    │    Petugas memeriksa
    │        │
    │    ┌───┴──────────┐
    │    │              │
    │ Keluarga sama   Bukan
    │    │              │
    │    ▼              ▼
    │ Gunakan        Buat keluarga
    │ existing       baru
    │    │              │
    └────┴──────┬───────┘
                ▼
        Simpan pembayaran
```

---

## 4.4 Tidak Ditemukan Kandidat

Jika tidak ditemukan keluarga yang berpotensi sama:

1. Sistem membuat pencatatan keluarga baru.
2. Sistem menghubungkan keluarga dengan lokasi.
3. Sistem menghubungkan keluarga dengan periode.
4. Sistem menyimpan pembayaran pertama.
5. Sistem memperbarui rekap lokasi.

---

## 4.5 Ditemukan Kandidat

Jika ditemukan satu atau lebih kandidat:

Sistem menampilkan informasi yang membantu Petugas membedakan keluarga.

Contoh:

```text
Kemungkinan keluarga sudah terdaftar:

1. Ahmad
   RT 02 / RW 01
   Pembayaran sebelumnya: 2 jiwa

2. Ahmad
   RT 03 / RW 01
   Pembayaran sebelumnya: 4 jiwa
```

Petugas kemudian memilih:

```text
[ Tambahkan ke keluarga ini ]

atau

[ Buat pencatatan keluarga baru ]
```

Sistem tidak boleh melakukan merge secara otomatis apabila terdapat ketidakpastian identitas.

---

# 5. Workflow 3 — Pembayaran Tambahan

## 5.1 Tujuan

Menambahkan pembayaran baru pada keluarga yang sebelumnya telah melakukan pembayaran.

## 5.2 Contoh Kasus

Hari pertama:

```text
Keluarga Ahmad

Pembayaran #1
2 jiwa
5 kg beras
```

Hari berikutnya:

```text
Pembayaran #2
1 jiwa
Pembayaran uang
```

Hasil:

```text
Keluarga Ahmad
│
├── Pembayaran #1
│   └── 2 jiwa / 5 kg
│
└── Pembayaran #2
    └── 1 jiwa / uang
```

Total:

```text
KK          : 1
Total jiwa  : 3
Beras       : 5 kg
Uang        : sesuai pembayaran #2
```

---

## 5.3 Workflow

```text
Petugas isi pembayaran
        │
        ▼
Klik SIMPAN
        │
        ▼
Sistem menemukan keluarga existing
        │
        ▼
Petugas konfirmasi
        │
        ▼
Sistem membuat PAYMENT BARU
        │
        ▼
Payment lama tetap disimpan
        │
        ▼
Hitung ulang total keluarga
        │
        ▼
Hitung ulang rekap lokasi
```

## 5.4 Rule

Pembayaran tambahan:

* tidak mengubah pembayaran lama;
* tidak menambah jumlah KK;
* menambah jumlah jiwa sesuai pembayaran baru;
* menambah total beras dan/atau uang sesuai pembayaran baru;
* tetap memiliki tanggal dan audit trail sendiri.

---

# 6. Workflow 4 — Pembayaran Campuran

## 6.1 Tujuan

Mengakomodasi pembayaran dalam satu pencatatan menggunakan beras dan uang sekaligus.

## 6.2 Contoh

```text
Keluarga Ahmad

Pembayaran:
Total Jiwa : 5

Beras:
3 jiwa
7,5 kg

Uang:
2 jiwa
Rp xxx.xxx
```

## 6.3 Workflow

```text
Petugas memilih pembayaran
        │
        ▼
Isi Jiwa Beras
        │
        ▼
Isi Jumlah Beras
        │
        ▼
Isi Jiwa Uang
        │
        ▼
Isi Nominal Uang
        │
        ▼
Sistem validasi
        │
        ▼
Jiwa Beras + Jiwa Uang
      =
Total Jiwa?
    │
 ┌──┴──┐
Tidak  Ya
 │      │
 ▼      ▼
Tolak  Lanjut
```

Jika hanya menggunakan satu bentuk:

```text
Beras saja:
Jiwa beras = total jiwa
Jiwa uang = 0
```

atau:

```text
Uang saja:
Jiwa uang = total jiwa
Jiwa beras = 0
```

---

# 7. Workflow 5 — Perubahan Pembayaran

## 7.1 Tujuan

Memperbaiki pembayaran yang salah selama status rekap masih memperbolehkan perubahan.

## 7.2 Workflow

```text
Petugas membuka pembayaran
        │
        ▼
Sistem memeriksa status rekap
        │
        ▼
┌───────────────────────────┐
│ Status memperbolehkan edit?│
└─────────────┬─────────────┘
          ┌───┴───┐
         Tidak    Ya
          │        │
          ▼        ▼
       Ditolak   Edit data
                   │
                   ▼
                Validasi
                   │
                   ▼
                Simpan
                   │
                   ▼
              Audit perubahan
                   │
                   ▼
              Update rekap
```

## 7.3 Permission Berdasarkan Status

| Status Rekap        | Edit Pembayaran |
| ------------------- | :-------------: |
| `DRAFT`             |        ✅        |
| `SUBMITTED`         |        ❌        |
| `REVISION_REQUIRED` |        ✅        |
| `APPROVED`          |        ❌        |

---

# 8. Workflow 6 — Pembatalan Pembayaran

## 8.1 Tujuan

Membatalkan pembayaran yang tidak valid tanpa menghapus histori secara permanen.

## 8.2 Workflow

```text
Petugas memilih pembayaran
        │
        ▼
Pilih Batalkan
        │
        ▼
Sistem cek status rekap
        │
        ├── Tidak boleh
        │       ↓
        │    Ditolak
        │
        └── Boleh
                ↓
         Masukkan alasan
                ↓
            Konfirmasi
                ↓
         Tandai dibatalkan
                ↓
         Simpan audit trail
                ↓
          Hitung ulang rekap
```

## 8.3 Rule

Pembayaran yang dibatalkan:

* tetap berada pada histori;
* tidak ikut dihitung dalam total jiwa;
* tidak ikut dihitung dalam total beras;
* tidak ikut dihitung dalam total uang;
* tidak menghilangkan histori siapa yang membuat atau membatalkan pembayaran.

---

# 9. Workflow 7 — Pembentukan Rekap Lokasi

## 9.1 Tujuan

Menghasilkan ringkasan otomatis berdasarkan pembayaran valid pada satu lokasi dan periode.

## 9.2 Sumber Data

Rekap lokasi dihitung dari:

```text
Pencatatan keluarga
       │
       └── seluruh pembayaran VALID
```

Pembayaran yang dibatalkan tidak dihitung.

## 9.3 Perhitungan

Rekap minimal menghasilkan:

```text
Jumlah KK
Jumlah Jiwa
Jumlah Jiwa Beras
Total Beras
Jumlah Jiwa Uang
Total Uang
```

## 9.4 Jumlah KK

Satu keluarga hanya dihitung satu kali meskipun memiliki banyak pembayaran.

Contoh:

```text
Ahmad
├── Payment #1
├── Payment #2
└── Payment #3
```

Tetap dihitung:

```text
1 KK
```

bukan:

```text
3 KK
```

---

# 10. Workflow 8 — Pengiriman Rekap Lokasi

## 10.1 Tujuan

Mengirim hasil pendataan lokasi kepada Admin Desa.

## 10.2 Preconditions

* Petugas telah login.
* Lokasi memiliki data valid.
* Rekap berstatus `DRAFT` atau kondisi revisi yang telah selesai diperbaiki.
* Petugas memiliki kewenangan terhadap lokasi.

## 10.3 Workflow

```text
Petugas buka Rekap Lokasi
        │
        ▼
Sistem hitung rekap
        │
        ▼
Petugas memeriksa
        │
        ▼
Klik Kirim Rekap
        │
        ▼
Sistem meminta konfirmasi
        │
        ▼
Petugas konfirmasi
        │
        ▼
Status → SUBMITTED
        │
        ▼
Transaksi dikunci
        │
        ▼
Catat:
- submitted_by
- submitted_at
```

## 10.4 Rekap Kosong

Jika tidak terdapat data valid:

```text
Kirim Rekap
     ↓
Sistem mendeteksi
tidak ada data
     ↓
Pengiriman ditolak
```

---

# 11. Workflow 9 — Pemeriksaan Rekap

## 11.1 Actor

Admin Desa.

## 11.2 Workflow

```text
Rekap SUBMITTED
        │
        ▼
Admin membuka rekap
        │
        ▼
Lihat ringkasan
        │
        ▼
Jika diperlukan:
lihat detail keluarga/pembayaran
        │
        ▼
Admin menentukan
        │
   ┌────┴───────────┐
   │                │
Sesuai          Perlu diperbaiki
   │                │
   ▼                ▼
Setujui         Minta Revisi
```

---

# 12. Workflow 10 — Permintaan Revisi

## 12.1 Tujuan

Mengembalikan rekap kepada Petugas ketika terdapat data yang perlu diperbaiki.

## 12.2 Workflow

```text
SUBMITTED
    │
    ▼
Admin memilih
Minta Revisi
    │
    ▼
Masukkan alasan
    │
    ▼
Alasan kosong?
 │
 ├── Ya → Ditolak
 │
 └── Tidak
       ↓
REVISION_REQUIRED
       ↓
Transaksi dibuka
untuk perbaikan
       ↓
Petugas melihat
alasan revisi
```

---

# 13. Workflow 11 — Perbaikan dan Pengiriman Ulang

## 13.1 Workflow

```text
REVISION_REQUIRED
        │
        ▼
Petugas membaca
alasan revisi
        │
        ▼
Petugas memperbaiki:
- keluarga
- pembayaran
- pembatalan
sesuai kebutuhan
        │
        ▼
Sistem memperbarui rekap
        │
        ▼
Petugas memeriksa ulang
        │
        ▼
Kirim Ulang
        │
        ▼
SUBMITTED
        │
        ▼
Data dikunci kembali
        │
        ▼
Admin periksa ulang
```

Perbaikan data tidak otomatis berarti rekap disetujui.

Admin tetap harus melakukan pemeriksaan kembali.

---

# 14. Workflow 12 — Persetujuan Rekap

## 14.1 Actor

Admin Desa.

## 14.2 Workflow

```text
SUBMITTED
     │
     ▼
Admin selesai memeriksa
     │
     ▼
Klik Setujui
     │
     ▼
Konfirmasi
     │
     ▼
APPROVED
     │
     ▼
Catat:
- approved_by
- approved_at
     │
     ▼
Masuk ke rekap resmi desa
```

## 14.3 Setelah APPROVED

Ketika rekap sudah `APPROVED`:

* Petugas tidak dapat mengubah pembayaran;
* Petugas tidak dapat menambahkan pembayaran;
* Petugas tidak dapat membatalkan pembayaran;
* rekap menjadi sumber data resmi desa.

Mekanisme perubahan administratif setelah approval belum ditentukan pada V1 dan masih termasuk keputusan pending.

---

# 15. State Machine Rekap Lokasi

Workflow status rekap secara keseluruhan:

```text
                ┌─────────────┐
                │    DRAFT    │
                └──────┬──────┘
                       │
                   Kirim Rekap
                       │
                       ▼
                ┌─────────────┐
                │  SUBMITTED  │
                └──────┬──────┘
                       │
              ┌────────┴────────┐
              │                 │
         Minta Revisi        Setujui
              │                 │
              ▼                 ▼
 ┌─────────────────────┐   ┌──────────┐
 │ REVISION_REQUIRED   │   │ APPROVED │
 └──────────┬──────────┘   └──────────┘
            │
       Perbaiki data
            │
            ▼
        Kirim ulang
            │
            └──────────────→ SUBMITTED
```

---

# 16. State Transition Table

| Current State       | Action                          | Actor   | Next State          | Data Editable |
| ------------------- | ------------------------------- | ------- | ------------------- | :-----------: |
| `DRAFT`             | Tambah/Edit/Batalkan pembayaran | Petugas | `DRAFT`             |       ✅       |
| `DRAFT`             | Kirim Rekap                     | Petugas | `SUBMITTED`         |       ❌       |
| `SUBMITTED`         | Periksa                         | Admin   | `SUBMITTED`         |       ❌       |
| `SUBMITTED`         | Minta Revisi                    | Admin   | `REVISION_REQUIRED` |       ✅       |
| `SUBMITTED`         | Setujui                         | Admin   | `APPROVED`          |       ❌       |
| `REVISION_REQUIRED` | Edit/Batalkan/Tambah pembayaran | Petugas | `REVISION_REQUIRED` |       ✅       |
| `REVISION_REQUIRED` | Kirim Ulang                     | Petugas | `SUBMITTED`         |       ❌       |
| `APPROVED`          | Normal Edit                     | Petugas | Tidak diizinkan     |       ❌       |

---

# 17. Workflow 13 — Rekap Desa

## 17.1 Tujuan

Menggabungkan seluruh rekap lokasi yang telah disetujui.

## 17.2 Workflow

```text
Masjid A
APPROVED ──────┐

Musholla A
APPROVED ──────┤

Musholla B
SUBMITTED ──X  │
               │
SD A           │
APPROVED ──────┤
               │
TK A           │
APPROVED ──────┘
               │
               ▼
         REKAP RESMI DESA
```

Rekap yang belum `APPROVED` tidak masuk dalam rekap resmi.

---

# 18. Rekap Desa — Perhitungan

Rekap resmi desa minimal mencakup:

```text
Jumlah lokasi approved
Jumlah KK
Jumlah jiwa
Jumlah jiwa beras
Total beras
Jumlah jiwa uang
Total uang
```

Sistem juga dapat menampilkan rincian berdasarkan:

* lokasi;
* jenis lokasi;
* periode.

---

# 19. Workflow 14 — Laporan

## 19.1 Workflow

```text
Admin / DKM
      │
      ▼
Pilih menu laporan
      │
      ▼
Pilih periode
      │
      ▼
Pilih jenis laporan
      │
      ├── Per lokasi
      │
      └── Seluruh desa
      │
      ▼
Sistem mengambil
data sesuai aturan
      │
      ▼
Generate laporan
      │
      ▼
Preview
      │
      ▼
Export
  ┌───┴───┐
 PDF     Excel
```

Proses laporan tidak boleh mengubah data sumber.

---

# 20. Workflow 15 — Penutupan Periode

## 20.1 Tujuan

Menutup periode pendataan setelah proses zakat pada tahun tersebut selesai.

## 20.2 Workflow Konseptual

```text
Periode ACTIVE
      │
      ▼
Admin memilih
Tutup Periode
      │
      ▼
Sistem mengecek
status lokasi
      │
      ▼
Tampilkan ringkasan:
- Approved
- Submitted
- Revision Required
- Draft
      │
      ▼
Admin konfirmasi
      │
      ▼
Periode CLOSED
```

## 20.3 Dampak Periode CLOSED

Setelah periode ditutup:

* pendataan baru tidak dapat dilakukan secara normal;
* data historis tetap dapat dilihat;
* laporan tetap dapat dihasilkan;
* periode baru tidak mengubah data periode lama.

Aturan mengenai apakah periode boleh ditutup ketika masih terdapat lokasi yang belum `APPROVED` akan ditentukan lebih lanjut pada tahap System Design atau setelah kebutuhan administratif dikonfirmasi.

---

# 21. Transaction Editability Matrix

| Kondisi                 | Tambah Pembayaran | Edit Pembayaran | Batalkan Pembayaran |
| ----------------------- | :---------------: | :-------------: | :-----------------: |
| Periode aktif + `DRAFT` |         ✅         |        ✅        |          ✅          |
| `SUBMITTED`             |         ❌         |        ❌        |          ❌          |
| `REVISION_REQUIRED`     |         ✅         |        ✅        |          ✅          |
| `APPROVED`              |         ❌         |        ❌        |          ❌          |
| Periode `CLOSED`        |         ❌         |        ❌        |          ❌          |

Pengecualian administratif dapat ditambahkan di masa depan apabila memang dibutuhkan.

---

# 22. Audit Events

Workflow penting harus menghasilkan audit event.

Minimal:

| Event                      | Audit |
| -------------------------- | :---: |
| Membuat keluarga           |   ✅   |
| Membuat pembayaran         |   ✅   |
| Mengubah pembayaran        |   ✅   |
| Membatalkan pembayaran     |   ✅   |
| Mengubah konfigurasi zakat |   ✅   |
| Kirim rekap                |   ✅   |
| Minta revisi               |   ✅   |
| Kirim ulang                |   ✅   |
| Approve rekap              |   ✅   |
| Menonaktifkan pengguna     |   ✅   |
| Mengubah role              |   ✅   |
| Menonaktifkan lokasi       |   ✅   |

Audit event harus dapat mengidentifikasi minimal:

* actor;
* action;
* timestamp;
* objek yang terdampak.

---

# 23. Error and Failure Handling

Workflow harus mempertimbangkan kegagalan yang mungkin terjadi.

## 23.1 Validasi Gagal

Sistem tidak menyimpan perubahan dan menampilkan informasi kepada pengguna.

## 23.2 Gangguan Jaringan Saat Menyimpan

Sistem harus menghindari kondisi di mana pengguna tidak mengetahui apakah data telah tersimpan.

Implementasi detail seperti idempotency atau request identifier akan dipertimbangkan pada API Design.

## 23.3 Gangguan Saat Submit Rekap

Status rekap tidak boleh berubah menjadi `SUBMITTED` apabila proses pengiriman belum berhasil disimpan oleh server.

## 23.4 Export Gagal

Kegagalan membuat laporan tidak boleh mengubah data utama.

---

# 24. Concurrency Consideration

Sistem harus mempertimbangkan kemungkinan lebih dari satu pengguna melakukan operasi pada data yang berkaitan.

Contoh:

```text
Petugas sedang mengedit data

pada saat yang sama

Admin mencoba melakukan tindakan terhadap rekap.
```

Backend harus menjadi sumber kebenaran terakhir mengenai:

* status rekap;
* hak perubahan;
* validitas operasi.

UI tidak boleh menjadi satu-satunya mekanisme penguncian.

Detail mekanisme concurrency akan ditentukan pada System Design dan Database Design.

---

# 25. Authorization During Workflow

Sistem harus memeriksa authorization pada setiap operasi penting.

Contoh:

```text
Petugas Musholla A
↓
mencoba mengubah data Musholla B
↓
DITOLAK
```

atau:

```text
DKM
↓
mencoba approve rekap
↓
DITOLAK
```

Authorization harus diterapkan pada backend.

---

# 26. Workflow dan Rekap Otomatis

Rekap tidak boleh bergantung pada input total manual.

Contoh yang tidak diperbolehkan:

```text
Petugas mengetik:

Total KK = 120
Total Jiwa = 350
Total Beras = 875 kg
```

Sebagai sumber utama rekap.

Yang benar:

```text
Payment #1 ─┐
Payment #2 ─┤
Payment #3 ─┤
...         ├──→ Sistem menghitung otomatis
Payment #N ─┘
                  ↓
             Rekap Lokasi
```

Hal ini menjaga integritas data dan memungkinkan rekap dihitung ulang apabila pembayaran berubah.

---

# 27. Workflow Summary

Workflow utama sistem dapat diringkas menjadi:

```text
WARGA DATANG
     │
     ▼
PETUGAS INPUT PEMBAYARAN
     │
     ▼
VALIDASI
     │
     ▼
DETEKSI KELUARGA
     │
     ├── Existing → Tambah Payment
     │
     └── Baru     → Buat Family + Payment
     │
     ▼
REKAP LOKASI OTOMATIS
     │
     ▼
DRAFT
     │
     ▼
KIRIM
     │
     ▼
SUBMITTED
     │
 ┌───┴──────────────┐
 │                  │
 ▼                  ▼
REVISION_REQUIRED  APPROVED
 │                  │
 ▼                  │
PERBAIKAN            │
 │                  │
 ▼                  │
SUBMITTED ──────────►│
                     ▼
                REKAP DESA
                     │
                     ▼
                  LAPORAN
```

---

# 28. Keputusan yang Sudah Dikunci

Pada Workflow Design V1, keputusan berikut dianggap telah disepakati:

1. Petugas tidak perlu mencari keluarga sebelum mengisi pembayaran.
2. Sistem melakukan deteksi keluarga secara otomatis setelah Petugas memilih Simpan.
3. Sistem tidak boleh menggabungkan keluarga yang belum pasti sama secara otomatis.
4. Petugas mengonfirmasi kandidat keluarga apabila ditemukan kemungkinan data yang sama.
5. Satu keluarga dapat memiliki banyak pembayaran.
6. Pembayaran baru tidak menimpa pembayaran sebelumnya.
7. Pembayaran beras dan uang dapat digunakan sekaligus.
8. Jumlah jiwa berdasarkan bentuk pembayaran harus konsisten dengan total jiwa pembayaran.
9. Satu keluarga dihitung satu KK meskipun memiliki banyak pembayaran.
10. Rekap lokasi dihitung otomatis dari pembayaran valid.
11. Status workflow diterapkan pada Rekap Lokasi.
12. `SUBMITTED` mengunci perubahan normal.
13. `REVISION_REQUIRED` membuka kembali data untuk perbaikan.
14. `APPROVED` mengunci data dari perubahan normal.
15. Hanya Rekap Lokasi `APPROVED` yang masuk ke rekap resmi desa.
16. Pembatalan data tidak menghapus histori secara permanen.

---

# 29. Keputusan yang Masih Pending

## WF-PENDING-001 — Admin Override

Belum ditentukan apakah Admin Desa dapat:

* membuat pembayaran;
* mengubah pembayaran;
* membatalkan pembayaran

secara langsung.

---

## WF-PENDING-002 — Koreksi Setelah APPROVED

Belum ditentukan bagaimana proses apabila terdapat kesalahan setelah rekap berstatus `APPROVED`.

Kemungkinan:

* reopen;
* administrative correction;
* revision baru;
* approval ulang.

---

## WF-PENDING-003 — Penutupan Periode dengan Rekap Belum Selesai

Belum ditentukan apakah Admin diperbolehkan menutup periode apabila masih terdapat lokasi dengan status:

* `DRAFT`;
* `SUBMITTED`;
* `REVISION_REQUIRED`.

---

# 30. Keterkaitan dengan Dokumen Lain

Workflow Design harus tetap konsisten dengan:

* `04-functional-requirements.md`
* `05-non-functional-requirements.md`
* `06-use-case-analysis.md`
* `07-business-rules.md`
* `09-requirement-traceability.md`

Perubahan terhadap workflow yang memengaruhi requirement atau business rule harus dicatat pada `change-log.md`.

---

# 31. Kesimpulan

Workflow V1 Sistem Pendataan dan Rekapitulasi Zakat Fitrah Desa berpusat pada alur:

Pendataan Pembayaran → Deteksi Keluarga → Penyimpanan Riwayat Pembayaran → Rekap Lokasi → Pengiriman → Pemeriksaan → Revisi/Persetujuan → Rekap Desa → Laporan.

Satu keluarga dapat memiliki lebih dari satu pembayaran dan setiap pembayaran dipertahankan sebagai histori terpisah.

Sistem melakukan deteksi keluarga secara otomatis agar proses input tetap sederhana bagi Petugas Lokasi.

Rekap Lokasi menggunakan state:

`DRAFT → SUBMITTED → REVISION_REQUIRED / APPROVED`

dan hanya rekap yang telah `APPROVED` yang menjadi bagian dari rekap resmi desa.

Workflow ini menjadi dasar untuk tahap berikutnya yaitu System Design.
