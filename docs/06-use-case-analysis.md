# Use Case Analysis

## 1. Tujuan Dokumen

Dokumen ini mendefinisikan interaksi antara pengguna dengan Sistem Pendataan dan Rekapitulasi Zakat Fitrah Desa.

Use case digunakan untuk menjelaskan:

* siapa yang menggunakan sistem;
* tujuan masing-masing pengguna;
* fungsi yang dapat dilakukan;
* kondisi sebelum proses dilakukan;
* alur utama proses;
* kemungkinan alur alternatif atau kegagalan;
* kondisi setelah proses selesai;
* keterkaitan dengan Functional Requirements dan Business Rules.

Dokumen ini merupakan living document dan dapat diperbarui apabila terdapat perubahan kebutuhan selama proses desain, implementasi, testing, atau evaluasi sistem.

---

# 2. Actors

## 2.1 Petugas Lokasi

Petugas Lokasi merupakan pengguna yang bertanggung jawab melakukan pendataan zakat fitrah pada lokasi tertentu.

Lokasi dapat berupa:

* Masjid
* Musholla
* SD
* TK

Petugas Lokasi hanya dapat mengelola data yang berada pada lokasi yang ditugaskan kepadanya.

### Tanggung Jawab Utama

Petugas Lokasi bertanggung jawab untuk:

* melakukan login;
* mencatat pembayaran zakat fitrah;
* melihat data yang telah dicatat;
* memperbaiki data sesuai kewenangan;
* membatalkan data yang salah sesuai aturan;
* melihat rekap lokasi;
* mengirim rekap lokasi kepada Pemerintah Desa;
* melihat status pengumpulan data.

---

## 2.2 Admin Desa

Admin Desa merupakan pengguna yang memiliki kewenangan administratif pada tingkat desa.

### Tanggung Jawab Utama

Admin Desa bertanggung jawab untuk:

* mengelola pengguna;
* mengelola lokasi pembayaran zakat;
* menugaskan petugas;
* mengelola periode zakat fitrah;
* mengelola ketentuan zakat fitrah;
* melihat data seluruh lokasi;
* memeriksa rekap lokasi;
* meminta perbaikan;
* menyetujui rekap;
* melihat rekap desa;
* menghasilkan laporan;
* melihat status pengumpulan;
* melihat audit log.

---

## 2.3 DKM

DKM merupakan pengguna yang menggunakan sistem terutama untuk melihat hasil rekapitulasi dan laporan.

DKM pada V1 tidak memiliki kewenangan untuk mengubah data operasional zakat.

### Tanggung Jawab Utama

DKM dapat:

* melakukan login;
* melihat dashboard;
* melihat rekap lokasi;
* melihat rekap desa;
* melihat laporan;
* mengunduh laporan sesuai kewenangan.

---

## 2.4 Warga / Muzakki

Warga merupakan business actor, tetapi bukan pengguna langsung sistem pada V1.

Warga menunaikan zakat fitrah melalui Petugas Lokasi.

Data pembayaran warga kemudian dicatat oleh Petugas Lokasi ke dalam sistem.

Dengan demikian, warga tidak diwajibkan:

* memiliki akun;
* melakukan login;
* menggunakan aplikasi secara langsung.

---

# 3. Daftar Use Case

| ID     | Use Case                      | Primary Actor                   |
| ------ | ----------------------------- | ------------------------------- |
| UC-001 | Login                         | Petugas, Admin Desa, DKM        |
| UC-002 | Logout                        | Petugas, Admin Desa, DKM        |
| UC-003 | Kelola Pengguna               | Admin Desa                      |
| UC-004 | Kelola Role dan Hak Akses     | Admin Desa                      |
| UC-005 | Kelola Lokasi                 | Admin Desa                      |
| UC-006 | Menugaskan Petugas            | Admin Desa                      |
| UC-007 | Kelola Periode Zakat          | Admin Desa                      |
| UC-008 | Kelola Ketentuan Zakat Fitrah | Admin Desa                      |
| UC-009 | Catat Zakat Fitrah            | Petugas Lokasi                  |
| UC-010 | Lihat Data Zakat              | Petugas Lokasi, Admin Desa      |
| UC-011 | Ubah Data Zakat               | Petugas Lokasi, Admin Desa      |
| UC-012 | Batalkan Data Zakat           | Petugas Lokasi, Admin Desa      |
| UC-013 | Kirim Rekap Lokasi            | Petugas Lokasi                  |
| UC-014 | Periksa Rekap Lokasi          | Admin Desa                      |
| UC-015 | Minta Perbaikan Rekap         | Admin Desa                      |
| UC-016 | Setujui Rekap Lokasi          | Admin Desa                      |
| UC-017 | Lihat Rekap Lokasi            | Petugas Lokasi, Admin Desa, DKM |
| UC-018 | Lihat Rekap Desa              | Admin Desa, DKM                 |
| UC-019 | Generate Laporan              | Admin Desa, DKM                 |
| UC-020 | Export Laporan                | Admin Desa, DKM                 |
| UC-021 | Lihat Status Pengumpulan      | Petugas Lokasi, Admin Desa      |
| UC-022 | Lihat Audit Log               | Admin Desa                      |

---

# 4. Use Case Specifications

# UC-001 — Login

## Primary Actor

* Petugas Lokasi
* Admin Desa
* DKM

## Goal

Memungkinkan pengguna yang terdaftar masuk ke dalam sistem sesuai hak aksesnya.

## Preconditions

* Pengguna telah memiliki akun.
* Akun pengguna berstatus aktif.

## Trigger

Pengguna membuka halaman login.

## Main Flow

1. Sistem menampilkan halaman login.
2. Pengguna memasukkan username atau email.
3. Pengguna memasukkan password.
4. Pengguna memilih tombol Login.
5. Sistem memvalidasi data login.
6. Sistem memverifikasi status akun.
7. Sistem menentukan role pengguna.
8. Sistem membuat sesi autentikasi.
9. Sistem mengarahkan pengguna ke halaman sesuai role.

## Alternative Flow

### AF-01 — Kredensial Tidak Valid

1. Username/email atau password tidak sesuai.
2. Sistem menolak proses login.
3. Sistem menampilkan pesan bahwa kredensial tidak valid.

### AF-02 — Akun Tidak Aktif

1. Kredensial pengguna benar.
2. Sistem menemukan bahwa akun telah dinonaktifkan.
3. Sistem menolak akses.
4. Sistem menampilkan informasi bahwa akun tidak aktif.

## Postconditions

* Pengguna memiliki sesi autentikasi aktif.
* Pengguna dapat menggunakan fitur sesuai role dan kewenangan.

## Related Requirements

* FR-001
* FR-004
* NFR-001
* NFR-002
* NFR-003

## Related Business Rules

* BR-025

---

# UC-002 — Logout

## Primary Actor

* Petugas Lokasi
* Admin Desa
* DKM

## Goal

Mengakhiri sesi pengguna pada sistem.

## Preconditions

Pengguna telah login.

## Trigger

Pengguna memilih menu Logout.

## Main Flow

1. Pengguna memilih Logout.
2. Sistem mengakhiri sesi autentikasi pengguna.
3. Sistem menghapus atau membatalkan kredensial sesi yang digunakan.
4. Sistem mengarahkan pengguna ke halaman login.

## Postconditions

Pengguna tidak lagi memiliki sesi autentikasi aktif.

## Related Requirements

* FR-002

---

# UC-003 — Kelola Pengguna

## Primary Actor

Admin Desa

## Goal

Mengelola akun pengguna yang dapat mengakses sistem.

## Preconditions

* Admin Desa telah login.
* Admin Desa memiliki hak pengelolaan pengguna.

## Trigger

Admin Desa membuka menu Manajemen Pengguna.

## Main Flow

1. Sistem menampilkan daftar pengguna.
2. Admin dapat melihat informasi pengguna.
3. Admin dapat memilih Tambah Pengguna.
4. Admin memasukkan data pengguna.
5. Admin menentukan role pengguna.
6. Admin menentukan status akun.
7. Jika diperlukan, Admin menentukan lokasi penugasan.
8. Sistem melakukan validasi.
9. Sistem menyimpan data pengguna.
10. Sistem mencatat aktivitas penting pada audit log.

## Alternative Flow

### AF-01 — Data Tidak Lengkap

1. Admin tidak mengisi field wajib.
2. Sistem menolak penyimpanan.
3. Sistem menampilkan informasi field yang harus diperbaiki.

### AF-02 — Menonaktifkan Pengguna

1. Admin memilih pengguna.
2. Admin memilih Nonaktifkan Akun.
3. Sistem meminta konfirmasi.
4. Admin melakukan konfirmasi.
5. Sistem menonaktifkan akun tanpa menghapus histori aktivitas pengguna.

## Postconditions

Data pengguna telah dibuat atau diperbarui sesuai tindakan Admin.

## Related Requirements

* FR-003
* FR-004

## Related Business Rules

* BR-025

---

# UC-004 — Kelola Role dan Hak Akses

## Primary Actor

Admin Desa

## Goal

Menentukan role pengguna dan hak akses yang sesuai.

## Preconditions

* Admin Desa telah login.
* Pengguna yang akan diberikan role telah tersedia.

## Trigger

Admin membuka data pengguna atau pengaturan akses.

## Main Flow

1. Admin memilih pengguna.
2. Sistem menampilkan role pengguna saat ini.
3. Admin memilih role yang sesuai.
4. Sistem memvalidasi role.
5. Sistem menyimpan perubahan.
6. Hak akses pengguna disesuaikan berdasarkan role.
7. Sistem mencatat perubahan pada audit log.

## Postconditions

Role pengguna telah diperbarui.

## Related Requirements

* FR-004
* NFR-002

## Related Business Rules

* BR-005

---

# UC-005 — Kelola Lokasi

## Primary Actor

Admin Desa

## Goal

Mengelola lokasi penerimaan zakat fitrah.

## Preconditions

Admin Desa telah login.

## Trigger

Admin membuka menu Lokasi.

## Main Flow

1. Sistem menampilkan daftar lokasi.
2. Admin dapat menambahkan lokasi baru.
3. Admin memasukkan nama lokasi.
4. Admin menentukan jenis lokasi.
5. Admin memasukkan alamat atau keterangan jika diperlukan.
6. Admin menentukan status lokasi.
7. Sistem melakukan validasi.
8. Sistem menyimpan lokasi.

## Alternative Flow

### AF-01 — Menonaktifkan Lokasi

1. Admin memilih lokasi aktif.
2. Admin memilih Nonaktifkan.
3. Sistem meminta konfirmasi.
4. Admin mengonfirmasi.
5. Lokasi tidak dapat digunakan untuk pendataan baru.
6. Data historis lokasi tetap tersimpan.

## Postconditions

Data lokasi telah dibuat atau diperbarui.

## Related Requirements

* FR-005

## Related Business Rules

* BR-006
* BR-024

---

# UC-006 — Menugaskan Petugas

## Primary Actor

Admin Desa

## Goal

Menetapkan petugas pada lokasi penerimaan zakat tertentu.

## Preconditions

* Admin telah login.
* Akun Petugas Lokasi tersedia.
* Lokasi tersedia dan aktif.

## Trigger

Admin membuka pengaturan penugasan petugas.

## Main Flow

1. Admin memilih pengguna dengan role Petugas Lokasi.
2. Admin memilih lokasi penugasan.
3. Sistem memvalidasi pengguna dan lokasi.
4. Admin menyimpan penugasan.
5. Sistem menghubungkan petugas dengan lokasi.
6. Sistem menerapkan pembatasan akses berdasarkan lokasi.

## Postconditions

Petugas memiliki lokasi penugasan.

## Related Requirements

* FR-006

## Related Business Rules

* BR-005
* BR-006

---

# UC-007 — Kelola Periode Zakat

## Primary Actor

Admin Desa

## Goal

Mengatur periode pendataan zakat fitrah.

## Preconditions

Admin Desa telah login.

## Trigger

Admin membuka menu Periode Zakat.

## Main Flow

1. Sistem menampilkan daftar periode zakat.
2. Admin memilih Tambah Periode.
3. Admin memasukkan tahun/periode.
4. Admin menentukan tanggal mulai.
5. Admin menentukan tanggal selesai.
6. Admin menentukan status periode.
7. Sistem melakukan validasi.
8. Sistem menyimpan periode.

## Alternative Flow

### AF-01 — Mengaktifkan Periode

1. Admin memilih periode.
2. Admin memilih Aktifkan.
3. Sistem memeriksa konflik periode.
4. Sistem menetapkan periode sebagai aktif.

### AF-02 — Menutup Periode

1. Admin memilih periode aktif.
2. Admin memilih Tutup Periode.
3. Sistem meminta konfirmasi.
4. Sistem menutup periode.
5. Pendataan baru pada periode tersebut tidak lagi diperbolehkan sesuai kebijakan sistem.

## Postconditions

Periode zakat tersedia dengan status yang sesuai.

## Related Requirements

* FR-008
* FR-029

## Related Business Rules

* BR-007
* BR-008
* BR-023

---

# UC-008 — Kelola Ketentuan Zakat Fitrah

## Primary Actor

Admin Desa

## Goal

Menyimpan ketentuan zakat fitrah yang berlaku pada suatu periode.

## Preconditions

* Admin telah login.
* Periode zakat telah tersedia.

## Trigger

Admin membuka konfigurasi zakat fitrah.

## Main Flow

1. Admin memilih periode.
2. Sistem menampilkan konfigurasi periode tersebut.
3. Admin memasukkan besaran zakat beras per jiwa.
4. Admin memasukkan nominal uang per jiwa jika digunakan.
5. Admin menyimpan konfigurasi.
6. Sistem melakukan validasi.
7. Sistem menyimpan ketentuan untuk periode tersebut.
8. Sistem mencatat perubahan pada audit log.

## Alternative Flow

### AF-01 — Data Tidak Valid

1. Besaran yang dimasukkan bernilai negatif atau tidak valid.
2. Sistem menolak penyimpanan.
3. Sistem menampilkan pesan validasi.

## Postconditions

Ketentuan zakat tersimpan untuk periode yang dipilih.

## Related Requirements

* FR-007

## Related Business Rules

* BR-009
* BR-010

---

## UC-009 — Catat Zakat Fitrah

### Primary Actor

Petugas Lokasi

### Goal

Mencatat pembayaran zakat fitrah warga atau keluarga serta menghubungkan pembayaran tersebut dengan pencatatan keluarga yang sesuai.

### Preconditions

* Petugas telah login.
* Akun petugas aktif.
* Petugas memiliki lokasi penugasan.
* Periode pendataan zakat sedang aktif.

### Trigger

Warga datang menunaikan zakat dan Petugas memilih Tambah Pembayaran Zakat.

### Main Flow

1. Petugas membuka menu pendataan zakat.
2. Petugas memilih Tambah Pembayaran.
3. Sistem menampilkan formulir pembayaran.
4. Sistem menentukan lokasi berdasarkan penugasan Petugas.
5. Sistem menentukan periode zakat yang sedang aktif.
6. Petugas memasukkan nama kepala keluarga.
7. Petugas memasukkan informasi identifikasi keluarga yang diperlukan.
8. Petugas memasukkan jumlah jiwa yang dibayarkan.
9. Petugas memasukkan jumlah jiwa yang membayar menggunakan beras jika ada.
10. Petugas memasukkan jumlah beras jika ada.
11. Petugas memasukkan jumlah jiwa yang membayar menggunakan uang jika ada.
12. Petugas memasukkan nominal uang jika ada.
13. Petugas mengonfirmasi tanggal pembayaran.
14. Petugas memilih Simpan.
15. Sistem melakukan validasi data pembayaran.
16. Sistem melakukan pemeriksaan otomatis terhadap pencatatan keluarga yang telah tersedia pada periode dan konteks lokasi yang sama.
17. Jika tidak ditemukan keluarga yang sesuai, sistem membuat pencatatan keluarga baru.
18. Sistem menyimpan pembayaran sebagai riwayat pembayaran keluarga tersebut.
19. Sistem memperbarui perhitungan rekap lokasi.
20. Sistem mencatat aktivitas penting pada audit log.
21. Sistem menampilkan informasi bahwa pembayaran berhasil disimpan.

### Alternative Flow

#### AF-01 — Jumlah Jiwa Tidak Valid

1. Jumlah jiwa kurang dari satu atau tidak diisi.
2. Sistem menolak penyimpanan.
3. Sistem menampilkan pesan validasi.
4. Petugas memperbaiki data.

#### AF-02 — Bentuk Pembayaran Tidak Diisi

1. Pembayaran beras dan uang tidak memiliki nilai.
2. Sistem menolak penyimpanan.
3. Sistem meminta Petugas mengisi minimal satu bentuk pembayaran.

#### AF-03 — Jumlah Jiwa Pembayaran Tidak Konsisten

1. Total jiwa berdasarkan pembayaran beras dan uang tidak sama dengan jumlah jiwa pembayaran.
2. Sistem menolak penyimpanan.
3. Sistem menampilkan informasi ketidaksesuaian.
4. Petugas memperbaiki data.

Contoh:

Jumlah jiwa pembayaran: 5

Jiwa beras: 3
Jiwa uang: 2

Data valid karena 3 + 2 = 5.

#### AF-04 — Sistem Menemukan Kemungkinan Keluarga yang Sama

1. Setelah Petugas memilih Simpan, sistem menemukan satu atau lebih pencatatan keluarga yang berpotensi sama.
2. Sistem menampilkan kandidat keluarga yang ditemukan.
3. Sistem menampilkan informasi yang diperlukan untuk membantu Petugas membedakan keluarga.
4. Petugas memeriksa kandidat.

Jika salah satu kandidat merupakan keluarga yang sama:

5. Petugas memilih keluarga tersebut.
6. Petugas mengonfirmasi.
7. Sistem menambahkan pembayaran baru pada pencatatan keluarga tersebut.
8. Pembayaran sebelumnya tetap dipertahankan.

Jika kandidat bukan keluarga yang sama:

5. Petugas memilih Buat Pencatatan Keluarga Baru.
6. Sistem membuat pencatatan keluarga baru.
7. Sistem menyimpan pembayaran pada pencatatan baru.

#### AF-05 — Pembayaran Tambahan

1. Sistem menemukan pencatatan keluarga yang sebelumnya telah melakukan pembayaran.
2. Petugas mengonfirmasi bahwa keluarga tersebut sama.
3. Sistem tidak mengubah pembayaran sebelumnya.
4. Sistem membuat riwayat pembayaran baru pada keluarga tersebut.
5. Sistem menghitung ulang total keluarga dan rekap lokasi.

#### AF-06 — Nilai Beras Tidak Valid

1. Petugas memasukkan jumlah beras negatif.
2. Sistem menolak penyimpanan.
3. Sistem meminta Petugas memperbaiki jumlah beras.

#### AF-07 — Nominal Uang Tidak Valid

1. Petugas memasukkan nominal uang negatif.
2. Sistem menolak penyimpanan.
3. Sistem meminta Petugas memperbaiki nominal uang.

#### AF-08 — Periode Tidak Aktif

1. Petugas mencoba melakukan pendataan ketika periode pendataan tidak aktif.
2. Sistem menolak pencatatan.
3. Sistem menampilkan informasi bahwa periode zakat tidak sedang aktif.

### Postconditions

Jika proses berhasil:

* pencatatan keluarga tersedia;
* pembayaran tersimpan sebagai riwayat pembayaran;
* pembayaran tidak menimpa pembayaran sebelumnya;
* pembayaran terkait dengan lokasi Petugas;
* pembayaran terkait dengan periode zakat;
* rekap lokasi diperbarui;
* aktivitas penting dapat dilacak.

### Related Requirements

* FR-009
* FR-010
* FR-011
* FR-012
* FR-013
* FR-014
* FR-015
* FR-017
* FR-018

### Related Business Rules

* BR-001
* BR-002
* BR-003
* BR-004
* BR-005
* BR-006
* BR-007
* BR-008
* BR-062
* BR-063
* BR-064
* BR-065
* BR-066
* BR-067
* BR-068
* BR-069

---

# UC-010 — Lihat Data Zakat

## Primary Actor

* Petugas Lokasi
* Admin Desa

## Goal

Melihat data zakat yang telah dicatat.

## Preconditions

Pengguna telah login.

## Trigger

Pengguna membuka menu Data Zakat.

## Main Flow

1. Pengguna membuka daftar data zakat.
2. Sistem menentukan kewenangan pengguna.
3. Untuk Petugas Lokasi, sistem hanya menampilkan data lokasi petugas.
4. Untuk Admin Desa, sistem dapat menampilkan data seluruh lokasi.
5. Pengguna dapat melakukan pencarian.
6. Pengguna dapat menggunakan filter.
7. Pengguna memilih salah satu data.
8. Sistem menampilkan detail data.

## Postconditions

Tidak ada perubahan data.

## Related Requirements

* FR-016
* FR-039
* FR-040

## Related Business Rules

* BR-005

---

# UC-011 — Ubah Data Zakat

## Primary Actor

* Petugas Lokasi
* Admin Desa sesuai kewenangan

## Goal

Memperbaiki data zakat yang salah atau perlu diperbarui.

## Preconditions

* Pengguna telah login.
* Data zakat tersedia.
* Status data memungkinkan perubahan.
* Pengguna memiliki kewenangan terhadap data tersebut.

## Trigger

Pengguna memilih Edit pada data zakat.

## Main Flow

1. Pengguna membuka detail data.
2. Pengguna memilih Edit.
3. Sistem memeriksa hak akses.
4. Sistem memeriksa status data.
5. Sistem menampilkan formulir dengan data sebelumnya.
6. Pengguna melakukan perubahan.
7. Pengguna memilih Simpan.
8. Sistem melakukan validasi.
9. Sistem menyimpan perubahan.
10. Sistem memperbarui rekap.
11. Sistem mencatat nilai sebelum dan sesudah perubahan pada audit log.

## Alternative Flow

### AF-01 — Data Sudah Dikunci

1. Data berada dalam status yang tidak dapat diedit.
2. Sistem menolak perubahan.
3. Sistem menampilkan alasan.

### AF-02 — Validasi Gagal

1. Data baru tidak memenuhi aturan.
2. Sistem menolak penyimpanan.
3. Sistem meminta pengguna memperbaiki data.

## Postconditions

Data telah diperbarui dan perubahan dapat dilacak.

## Related Requirements

* FR-019
* FR-037
* FR-038

## Related Business Rules

* BR-015
* BR-016
* BR-017
* BR-021

---

# UC-012 — Batalkan Data Zakat

## Primary Actor

* Petugas Lokasi sesuai kewenangan
* Admin Desa sesuai kewenangan

## Goal

Membatalkan pencatatan zakat yang tidak seharusnya menjadi data aktif tanpa menghilangkan histori.

## Preconditions

* Pengguna telah login.
* Data tersedia.
* Pengguna memiliki hak akses.
* Status data memungkinkan pembatalan.

## Trigger

Pengguna memilih Batalkan Data.

## Main Flow

1. Pengguna membuka data zakat.
2. Pengguna memilih Batalkan.
3. Sistem meminta alasan pembatalan.
4. Pengguna memasukkan alasan.
5. Sistem meminta konfirmasi.
6. Pengguna mengonfirmasi.
7. Sistem mengubah status data menjadi status pembatalan yang ditentukan.
8. Data tidak lagi dihitung dalam rekap aktif.
9. Sistem menyimpan histori pembatalan.
10. Sistem memperbarui rekap.

## Alternative Flow

### AF-01 — Data Tidak Dapat Dibatalkan

1. Data telah berada dalam status final tertentu.
2. Sistem menolak pembatalan.
3. Sistem menampilkan informasi kepada pengguna.

## Postconditions

* Data tidak dihapus permanen.
* Data tetap tersedia sebagai histori.
* Rekap telah diperbarui.

## Related Requirements

* FR-012 terkait pengelolaan data
* FR-037
* FR-038

## Related Business Rules

* BR-021
* BR-026

---

# UC-013 — Kirim Rekap Lokasi

## Primary Actor

Petugas Lokasi

## Goal

Mengirim rekap zakat lokasi kepada Pemerintah Desa untuk diperiksa.

## Preconditions

* Petugas telah login.
* Petugas memiliki lokasi.
* Periode zakat aktif atau masih berada pada masa pengumpulan.
* Lokasi memiliki data yang dapat direkap.

## Trigger

Petugas memilih Kirim Rekap.

## Main Flow

1. Petugas membuka Rekap Lokasi.
2. Sistem menghitung jumlah KK.
3. Sistem menghitung jumlah jiwa.
4. Sistem menghitung total beras.
5. Sistem menghitung total uang.
6. Sistem menampilkan hasil rekap.
7. Petugas memeriksa hasil rekap.
8. Petugas memilih Kirim Rekap.
9. Sistem meminta konfirmasi.
10. Petugas mengonfirmasi.
11. Sistem mengubah status rekap menjadi `SUBMITTED`.
12. Sistem mencatat waktu dan pengguna yang melakukan pengiriman.
13. Sistem menampilkan informasi bahwa rekap berhasil dikirim.

## Alternative Flow

### AF-01 — Tidak Ada Data

1. Lokasi belum memiliki data zakat yang valid.
2. Sistem menolak pengiriman.
3. Sistem menampilkan informasi bahwa belum ada data untuk dikirim.

### AF-02 — Rekap Sudah Dikirim

1. Rekap telah berstatus `SUBMITTED`.
2. Sistem mencegah pengiriman ulang kecuali melalui alur revisi.

## Postconditions

Rekap lokasi berstatus `SUBMITTED` dan tersedia untuk diperiksa Admin Desa.

## Related Requirements

* FR-020
* FR-021
* FR-022
* FR-026

## Related Business Rules

* BR-011
* BR-012
* BR-013
* BR-014
* BR-015

---

# UC-014 — Periksa Rekap Lokasi

## Primary Actor

Admin Desa

## Goal

Memeriksa rekap yang telah dikirim oleh Petugas Lokasi.

## Preconditions

* Admin telah login.
* Terdapat rekap berstatus `SUBMITTED`.

## Trigger

Admin membuka daftar rekap masuk.

## Main Flow

1. Sistem menampilkan daftar rekap yang telah dikirim.
2. Admin memilih salah satu lokasi.
3. Sistem menampilkan ringkasan rekap.
4. Admin dapat melihat data detail jika diperlukan.
5. Admin memeriksa jumlah KK.
6. Admin memeriksa jumlah jiwa.
7. Admin memeriksa total beras.
8. Admin memeriksa total uang.
9. Admin menentukan apakah rekap dapat disetujui atau perlu diperbaiki.

## Postconditions

Rekap telah diperiksa tetapi status akhir bergantung pada tindakan Admin berikutnya.

## Related Requirements

* FR-023
* FR-026

## Related Business Rules

* BR-018

---

# UC-015 — Minta Perbaikan Rekap

## Primary Actor

Admin Desa

## Goal

Mengembalikan rekap kepada Petugas Lokasi apabila ditemukan kesalahan.

## Preconditions

* Admin telah login.
* Rekap berstatus `SUBMITTED`.

## Trigger

Admin menemukan data yang perlu diperbaiki.

## Main Flow

1. Admin membuka rekap lokasi.
2. Admin memilih Minta Perbaikan.
3. Sistem meminta alasan perbaikan.
4. Admin memasukkan alasan.
5. Admin mengonfirmasi.
6. Sistem mengubah status menjadi `REVISION_REQUIRED`.
7. Sistem menyimpan alasan perbaikan.
8. Petugas Lokasi dapat melihat status dan alasan revisi.

## Alternative Flow

### AF-01 — Alasan Kosong

1. Admin tidak memasukkan alasan.
2. Sistem menolak proses.
3. Sistem meminta Admin memasukkan alasan revisi.

## Postconditions

Rekap berstatus `REVISION_REQUIRED`.

## Related Requirements

* FR-025

## Related Business Rules

* BR-016
* BR-018
* BR-019

---

# UC-016 — Setujui Rekap Lokasi

## Primary Actor

Admin Desa

## Goal

Menetapkan rekap lokasi sebagai data yang telah diterima dan dapat digunakan dalam rekap resmi desa.

## Preconditions

* Admin telah login.
* Rekap berstatus `SUBMITTED`.
* Admin telah melakukan pemeriksaan.

## Trigger

Admin memilih Setujui Rekap.

## Main Flow

1. Admin membuka rekap.
2. Admin memilih Setujui.
3. Sistem meminta konfirmasi.
4. Admin mengonfirmasi.
5. Sistem mengubah status rekap menjadi `APPROVED`.
6. Sistem mencatat pengguna dan waktu persetujuan.
7. Sistem memasukkan data tersebut ke dalam perhitungan rekap resmi desa.
8. Sistem memperbarui dashboard desa.

## Postconditions

Rekap berstatus `APPROVED` dan digunakan dalam rekap resmi desa.

## Related Requirements

* FR-024
* FR-027

## Related Business Rules

* BR-017
* BR-018
* BR-020

---

# UC-017 — Lihat Rekap Lokasi

## Primary Actor

* Petugas Lokasi
* Admin Desa
* DKM

## Goal

Melihat ringkasan zakat fitrah berdasarkan lokasi.

## Preconditions

Pengguna telah login dan memiliki hak akses.

## Trigger

Pengguna membuka Rekap Lokasi.

## Main Flow

1. Sistem menentukan role pengguna.
2. Sistem menentukan lokasi yang boleh diakses.
3. Sistem mengambil data sesuai periode.
4. Sistem menghitung jumlah KK.
5. Sistem menghitung jumlah jiwa.
6. Sistem menghitung total beras.
7. Sistem menghitung total uang.
8. Sistem menampilkan hasil rekap.

## Postconditions

Tidak terdapat perubahan data.

## Related Requirements

* FR-026
* FR-028
* FR-029
* FR-030

## Related Business Rules

* BR-011
* BR-012
* BR-013
* BR-014

---

# UC-018 — Lihat Rekap Desa

## Primary Actor

* Admin Desa
* DKM

## Goal

Melihat rekapitulasi zakat fitrah seluruh desa.

## Preconditions

Pengguna telah login dan memiliki kewenangan.

## Trigger

Pengguna membuka menu Rekap Desa.

## Main Flow

1. Pengguna memilih periode.
2. Sistem mengambil data lokasi yang memenuhi ketentuan rekap resmi.
3. Sistem menghitung jumlah lokasi.
4. Sistem menghitung total KK.
5. Sistem menghitung total jiwa.
6. Sistem menghitung total beras.
7. Sistem menghitung total uang.
8. Sistem menampilkan ringkasan seluruh desa.
9. Pengguna dapat melihat rincian berdasarkan lokasi atau jenis lokasi.

## Postconditions

Tidak terdapat perubahan data.

## Related Requirements

* FR-027
* FR-028
* FR-029
* FR-031

## Related Business Rules

* BR-011
* BR-012
* BR-013
* BR-014
* BR-020

---

# UC-019 — Generate Laporan

## Primary Actor

* Admin Desa
* DKM

## Goal

Menghasilkan laporan rekapitulasi zakat berdasarkan data sistem.

## Preconditions

* Pengguna telah login.
* Pengguna memiliki kewenangan melihat laporan.
* Periode yang dipilih memiliki data.

## Trigger

Pengguna memilih menu Laporan.

## Main Flow

1. Pengguna memilih periode.
2. Pengguna memilih jenis laporan.
3. Sistem mengambil data yang sesuai.
4. Sistem menghitung rekap.
5. Sistem menyusun informasi laporan.
6. Sistem menampilkan preview laporan.

## Jenis Laporan V1

* Laporan rekap per lokasi.
* Laporan rekap seluruh desa.

## Postconditions

Laporan tersedia untuk dilihat atau diekspor.

## Related Requirements

* FR-033
* FR-034
* FR-035

## Related Business Rules

* BR-020

---

# UC-020 — Export Laporan

## Primary Actor

* Admin Desa
* DKM

## Goal

Mengunduh laporan dalam format dokumen yang dapat digunakan untuk administrasi dan arsip.

## Preconditions

Laporan telah berhasil dibuat.

## Trigger

Pengguna memilih Export atau Download.

## Main Flow

1. Pengguna membuka laporan.
2. Pengguna memilih format export.
3. Sistem menghasilkan file.
4. Sistem memastikan data laporan sesuai periode dan filter.
5. Sistem menyediakan file kepada pengguna.

## Format V1

* PDF
* Excel

## Alternative Flow

### AF-01 — Gagal Menghasilkan File

1. Sistem gagal membuat file.
2. Sistem tidak menghasilkan dokumen yang rusak.
3. Sistem menampilkan pesan kesalahan yang sesuai.
4. Error teknis dicatat pada log server.

## Postconditions

File laporan berhasil dihasilkan tanpa mengubah data utama sistem.

## Related Requirements

* FR-036
* NFR-018

---

# UC-021 — Lihat Status Pengumpulan

## Primary Actor

* Petugas Lokasi
* Admin Desa

## Goal

Mengetahui status proses pengumpulan rekap setiap lokasi.

## Preconditions

Pengguna telah login.

## Trigger

Pengguna membuka Dashboard atau Status Pengumpulan.

## Main Flow

### Untuk Petugas Lokasi

1. Sistem menampilkan status rekap lokasi petugas.
2. Sistem menampilkan apakah rekap masih draft, telah dikirim, perlu revisi, atau telah disetujui.
3. Jika terdapat revisi, sistem menampilkan keterangan yang relevan.

### Untuk Admin Desa

1. Sistem menampilkan seluruh lokasi.
2. Sistem menampilkan status masing-masing lokasi.
3. Admin dapat melihat lokasi yang belum mengirim.
4. Admin dapat melihat lokasi yang membutuhkan revisi.
5. Admin dapat melihat lokasi yang sudah disetujui.

## Postconditions

Tidak ada perubahan data.

## Related Requirements

* FR-022
* FR-032

---

# UC-022 — Lihat Audit Log

## Primary Actor

Admin Desa

## Goal

Melihat histori aktivitas penting yang terjadi pada sistem.

## Preconditions

Admin Desa telah login dan memiliki kewenangan melihat audit log.

## Trigger

Admin membuka menu Audit Log.

## Main Flow

1. Sistem menampilkan daftar aktivitas.
2. Admin dapat melihat waktu aktivitas.
3. Admin dapat melihat pengguna yang melakukan aktivitas.
4. Admin dapat melihat jenis tindakan.
5. Admin dapat melihat objek/data yang terdampak.
6. Jika berupa perubahan data, sistem dapat menampilkan informasi perubahan yang relevan.
7. Admin dapat melakukan pencarian atau filter jika diperlukan.

## Postconditions

Tidak terdapat perubahan terhadap audit log.

## Related Requirements

* FR-037
* FR-038
* NFR-013

## Related Business Rules

* BR-021
* BR-022

---

# 5. Authorization Matrix

| Use Case                      |         Petugas Lokasi         |        Admin Desa       | DKM |
| ----------------------------- | :----------------------------: | :---------------------: | :-: |
| UC-001 Login                  |                ✅               |            ✅            |  ✅  |
| UC-002 Logout                 |                ✅               |            ✅            |  ✅  |
| UC-003 Kelola Pengguna        |                ❌               |            ✅            |  ❌  |
| UC-004 Kelola Role/Hak Akses  |                ❌               |            ✅            |  ❌  |
| UC-005 Kelola Lokasi          |                ❌               |            ✅            |  ❌  |
| UC-006 Menugaskan Petugas     |                ❌               |            ✅            |  ❌  |
| UC-007 Kelola Periode         |                ❌               |            ✅            |  ❌  |
| UC-008 Kelola Ketentuan Zakat |                ❌               |            ✅            |  ❌  |
| UC-009 Catat Zakat            |                ✅               | Opsional/Admin Override |  ❌  |
| UC-010 Lihat Data Zakat       |         Lokasi sendiri         | Semua sesuai kewenangan |  ❌  |
| UC-011 Ubah Data              | Lokasi sendiri & sesuai status |    Sesuai kewenangan    |  ❌  |
| UC-012 Batalkan Data          | Lokasi sendiri & sesuai status |    Sesuai kewenangan    |  ❌  |
| UC-013 Kirim Rekap            |                ✅               |            ❌            |  ❌  |
| UC-014 Periksa Rekap          |                ❌               |            ✅            |  ❌  |
| UC-015 Minta Perbaikan        |                ❌               |            ✅            |  ❌  |
| UC-016 Setujui Rekap          |                ❌               |            ✅            |  ❌  |
| UC-017 Lihat Rekap Lokasi     |         Lokasi sendiri         |            ✅            |  ✅  |
| UC-018 Lihat Rekap Desa       |                ❌               |            ✅            |  ✅  |
| UC-019 Generate Laporan       |                ❌               |            ✅            |  ✅  |
| UC-020 Export Laporan         |                ❌               |            ✅            |  ✅  |
| UC-021 Status Pengumpulan     |         Lokasi sendiri         |       Semua lokasi      |  ❌  |
| UC-022 Audit Log              |                ❌               |            ✅            |  ❌  |

---

# 6. Hubungan Antar Use Case

Beberapa use case memiliki hubungan proses.

## Pendataan Zakat

`UC-009 Catat Zakat Fitrah`

dapat dilanjutkan dengan:

`UC-010 Lihat Data Zakat`

dan apabila diperlukan:

`UC-011 Ubah Data Zakat`

atau:

`UC-012 Batalkan Data Zakat`

---

## Submission Workflow

Alur utama pengiriman rekap adalah:

`UC-017 Lihat Rekap Lokasi`

→ `UC-013 Kirim Rekap Lokasi`

→ `UC-014 Periksa Rekap Lokasi`

Kemudian terdapat dua kemungkinan:

### Data Benar

`UC-016 Setujui Rekap Lokasi`

### Data Perlu Diperbaiki

`UC-015 Minta Perbaikan Rekap`

→ `UC-011 Ubah Data Zakat`

→ `UC-013 Kirim Rekap Lokasi`

---

## Reporting Workflow

Rekap yang telah memenuhi ketentuan digunakan dalam:

`UC-018 Lihat Rekap Desa`

→ `UC-019 Generate Laporan`

→ `UC-020 Export Laporan`

---

# 7. Status Rekap Konseptual

Pada tahap Use Case Analysis, sistem menggunakan status konseptual berikut:

| Status              | Deskripsi                                                         |
| ------------------- | ----------------------------------------------------------------- |
| `DRAFT`             | Data masih berada pada proses pendataan dan belum dikirim.        |
| `SUBMITTED`         | Rekap telah dikirim oleh Petugas Lokasi kepada Admin Desa.        |
| `REVISION_REQUIRED` | Admin Desa meminta perbaikan terhadap rekap.                      |
| `APPROVED`          | Rekap telah disetujui dan dapat digunakan dalam rekap resmi desa. |

Status tersebut masih bersifat konseptual.

Detail implementasi status akan didefinisikan lebih lanjut dalam Workflow Design dan Database Design.

---

# 8. Catatan Desain yang Belum Dikunci

Beberapa keputusan sengaja belum ditentukan secara final pada tahap Use Case Analysis.

## Admin Melakukan Input Zakat

Saat ini Admin Desa dapat dipertimbangkan memiliki override untuk memasukkan atau memperbaiki data dalam kondisi administratif tertentu.

Keputusan final akan ditentukan dalam Authorization Design.

## Pembayaran Beras dan Uang

V1 saat ini memungkinkan pencatatan beras, uang, atau keduanya.

Aturan ini dapat berubah apabila proses lapangan menunjukkan bahwa satu pencatatan hanya boleh menggunakan satu bentuk pembayaran.

## Identifikasi Keluarga

Saat ini pencatatan menggunakan nama kepala keluarga dan jumlah jiwa.

Belum ditetapkan bahwa satu kepala keluarga hanya boleh memiliki satu pencatatan dalam satu periode.

Hal ini harus dikonfirmasi berdasarkan praktik lapangan sebelum dibuat menjadi aturan unik pada database.

## Approval

Approval dilakukan terhadap rekap lokasi, bukan terhadap setiap transaksi secara individual.

Keputusan ini akan diperjelas pada Workflow Design.

## Penghapusan Data

Data zakat tidak direncanakan untuk dihapus permanen dalam proses normal.

Pembatalan atau soft delete akan dipertimbangkan agar histori tetap dapat dilacak.

---

# 9. Traceability

Setiap use case harus dapat dilacak ke Functional Requirement dan Business Rule yang relevan.

Dokumen `09-requirement-traceability.md` akan digunakan untuk mencatat keterkaitan antara:

Functional Requirement → Use Case → Business Rule → Design → API → Test Case.

Traceability akan terus diperbarui sepanjang pengembangan sistem.

---

# 10. Change Management

Use Case Analysis dapat diperbarui apabila ditemukan:

* kebutuhan baru;
* perubahan proses bisnis;
* perubahan aturan operasional;
* hasil observasi lapangan;
* feedback pengguna;
* perubahan saat implementasi;
* hasil testing.

Setiap perubahan penting harus dicatat pada `change-log.md`.

Perubahan harus memperhatikan dampaknya terhadap:

* Functional Requirements;
* Non-Functional Requirements;
* Business Rules;
* Workflow;
* Database;
* API;
* UI;
* Security;
* Test Case.

---

# 11. Kesimpulan

Use Case Analysis V1 mendefinisikan tiga pengguna utama sistem:

1. Petugas Lokasi sebagai pengguna operasional pendataan.
2. Admin Desa sebagai pengguna administratif dan pengelola rekap.
3. DKM sebagai pengguna informasi dan laporan.

Warga merupakan business actor dan tidak menggunakan sistem secara langsung pada V1.

Proses utama sistem berpusat pada:

Pendataan Zakat → Rekap Lokasi → Pengiriman → Pemeriksaan → Revisi/Persetujuan → Rekap Desa → Laporan.

Dokumen ini menjadi dasar untuk tahap berikutnya yaitu Business Rules final dan Workflow Design.
