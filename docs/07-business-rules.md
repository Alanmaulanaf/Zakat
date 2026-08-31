# Business Rules

## 1. Tujuan Dokumen

Dokumen ini mendefinisikan aturan bisnis yang mengatur perilaku Sistem Pendataan dan Rekapitulasi Zakat Fitrah Desa.

Business Rule menjelaskan aturan yang harus dipatuhi sistem terlepas dari implementasi teknis yang digunakan.

Dokumen ini menjadi dasar untuk:

* Workflow Design
* System Design
* Database Design
* API Design
* Authorization
* Validation
* Testing

Business Rule dapat diperbarui apabila ditemukan perubahan proses bisnis selama pengembangan atau setelah sistem digunakan.

---

# 2. Pendataan Zakat Fitrah

## BR-001 — Jumlah Jiwa Minimal

Setiap pencatatan zakat fitrah harus memiliki jumlah jiwa minimal 1.

Nilai berikut tidak diperbolehkan:

* 0
* angka negatif
* nilai kosong

---

## BR-002 — Bentuk Pembayaran

Setiap pencatatan zakat fitrah harus memiliki minimal satu bentuk pembayaran.

Bentuk pembayaran pada V1 adalah:

* Beras
* Uang

Satu pencatatan saat ini dapat menggunakan:

* beras saja;
* uang saja;
* beras dan uang.

Aturan ini dapat berubah apabila praktik operasional di lapangan menetapkan bahwa satu pencatatan hanya boleh menggunakan satu bentuk pembayaran.

---

## BR-003 — Jumlah Beras

Jumlah beras tidak boleh bernilai negatif.

Apabila pembayaran tidak menggunakan beras, nilai beras dapat bernilai kosong atau nol sesuai implementasi data yang dipilih pada Database Design.

---

## BR-004 — Nominal Uang

Nominal uang tidak boleh bernilai negatif.

Apabila pembayaran tidak menggunakan uang, nilai nominal dapat bernilai kosong atau nol sesuai implementasi data yang dipilih pada Database Design.

---

## BR-005 — Nama Kepala Keluarga

Setiap pencatatan zakat fitrah harus memiliki informasi nama kepala keluarga atau identitas keluarga yang ditetapkan oleh sistem.

---

## BR-006 — Lokasi Pendataan

Setiap pencatatan zakat fitrah harus terkait dengan satu lokasi penerimaan zakat.

Lokasi dapat berupa:

* Masjid
* Musholla
* SD
* TK

---

## BR-007 — Periode Zakat

Setiap pencatatan zakat fitrah harus terkait dengan satu periode zakat.

Data dari periode yang berbeda tidak boleh tercampur dalam satu rekap.

---

## BR-008 — Periode Aktif

Petugas Lokasi hanya dapat membuat pencatatan baru pada periode zakat yang sedang aktif.

Pengecualian hanya dapat dilakukan melalui kewenangan administratif yang secara khusus ditetapkan kemudian.

---

## BR-009 — Tanggal Pembayaran

Tanggal pembayaran harus tersimpan pada setiap pencatatan zakat.

Tanggal pembayaran harus berada dalam konteks periode zakat yang sesuai, kecuali terdapat alasan administratif yang sah.

---

# 3. Ketentuan Zakat

## BR-010 — Besaran Zakat Tidak Di-Hard-Code

Besaran zakat fitrah per jiwa tidak boleh ditanam secara permanen di dalam source code.

Besaran zakat harus dapat dikonfigurasi berdasarkan periode.

---

## BR-011 — Ketentuan Berdasarkan Periode

Setiap periode dapat memiliki ketentuan zakat fitrah sendiri.

Contoh konfigurasi dapat mencakup:

* jumlah beras per jiwa;
* nominal uang per jiwa.

---

## BR-012 — Data Historis Ketentuan

Perubahan ketentuan pada periode baru tidak boleh mengubah ketentuan maupun data historis periode sebelumnya.

---

# 4. Pengguna dan Hak Akses

## BR-013 — Pengguna Harus Terautentikasi

Fitur sistem yang bersifat internal hanya dapat digunakan oleh pengguna yang telah melakukan autentikasi.

---

## BR-014 — Akses Berdasarkan Role

Hak akses pengguna ditentukan berdasarkan role.

Role V1:

* Petugas Lokasi
* Admin Desa
* DKM

---

## BR-015 — Pembatasan Lokasi Petugas

Petugas Lokasi hanya dapat melihat dan mengelola data pada lokasi yang ditugaskan kepadanya.

Petugas Lokasi tidak boleh mengakses data operasional lokasi lain.

---

## BR-016 — Hak Akses Admin Desa

Admin Desa dapat melihat data seluruh lokasi sesuai kewenangan administratifnya.

Perubahan terhadap data operasional oleh Admin harus tetap tercatat dalam audit log.

---

## BR-017 — Hak Akses DKM

DKM hanya memiliki akses terhadap informasi rekap dan laporan yang diizinkan.

DKM pada V1 tidak dapat:

* membuat data zakat;
* mengubah transaksi zakat;
* membatalkan transaksi;
* mengelola pengguna;
* mengelola lokasi.

---

## BR-018 — Akun Tidak Aktif

Pengguna dengan akun tidak aktif tidak dapat melakukan login.

Data historis yang pernah dibuat pengguna tersebut tetap dipertahankan.

---

# 5. Lokasi Zakat

## BR-019 — Lokasi Aktif

Hanya lokasi dengan status aktif yang dapat digunakan untuk pencatatan baru.

---

## BR-020 — Lokasi Nonaktif

Lokasi yang telah dinonaktifkan tidak dapat digunakan untuk pendataan baru.

Data historis lokasi tetap dapat dipertahankan dan dilihat sesuai kewenangan.

---

## BR-021 — Penugasan Petugas

Petugas harus memiliki lokasi penugasan sebelum dapat melakukan pendataan zakat.

---

# 6. Perubahan dan Pembatalan Data

## BR-022 — Data Dapat Diubah Sebelum Dikunci

Data zakat dapat diubah oleh Petugas Lokasi selama rekap lokasi belum berada pada kondisi yang mengunci perubahan.

---

## BR-023 — Rekap SUBMITTED Mengunci Perubahan Normal

Ketika rekap lokasi telah berstatus `SUBMITTED`, Petugas Lokasi tidak dapat mengubah transaksi secara bebas.

Perubahan hanya dapat dilakukan setelah rekap dikembalikan melalui proses revisi atau melalui kewenangan administratif yang sah.

---

## BR-024 — Data Dapat Diperbaiki Saat Revisi

Jika rekap lokasi berstatus `REVISION_REQUIRED`, Petugas Lokasi dapat melakukan perbaikan terhadap data yang terkait.

Setelah perbaikan selesai, rekap harus dikirim ulang.

---

## BR-025 — Rekap APPROVED Mengunci Data

Jika rekap lokasi telah berstatus `APPROVED`, Petugas Lokasi tidak dapat mengubah transaksi yang termasuk dalam rekap tersebut.

Perubahan setelah approval membutuhkan mekanisme administratif khusus yang akan ditentukan apabila kebutuhan tersebut muncul.

---

## BR-026 — Pembatalan Bukan Penghapusan Permanen

Pembatalan transaksi tidak boleh menghilangkan histori transaksi secara permanen.

Data yang dibatalkan harus tetap dapat dilacak.

---

## BR-027 — Alasan Pembatalan

Setiap pembatalan transaksi harus memiliki alasan yang tercatat.

---

# 7. Rekapitulasi Lokasi

## BR-028 — Jumlah KK

Jumlah KK pada rekap lokasi dihitung berdasarkan jumlah pencatatan keluarga yang valid dan tidak dibatalkan.

Jumlah KK tidak dihitung berdasarkan jumlah jiwa.

---

## BR-029 — Total Jiwa

Total jiwa pada rekap lokasi merupakan penjumlahan jumlah jiwa dari seluruh pencatatan yang valid dan termasuk dalam periode tersebut.

---

## BR-030 — Total Beras

Total beras pada rekap lokasi merupakan penjumlahan seluruh jumlah beras dari pencatatan yang valid.

---

## BR-031 — Total Uang

Total uang pada rekap lokasi merupakan penjumlahan seluruh nominal uang dari pencatatan yang valid.

---

## BR-032 — Data Dibatalkan Tidak Masuk Rekap

Transaksi yang telah dibatalkan tidak boleh dihitung dalam:

* jumlah KK;
* jumlah jiwa;
* total beras;
* total uang.

---

## BR-033 — Rekap Berdasarkan Lokasi dan Periode

Satu rekap lokasi harus merepresentasikan:

* satu lokasi;
* satu periode zakat.

Data dari lokasi atau periode lain tidak boleh masuk ke rekap tersebut.

---

# 8. Workflow Rekap

## BR-034 — Status Rekap

Rekap lokasi pada V1 memiliki status konseptual:

* `DRAFT`
* `SUBMITTED`
* `REVISION_REQUIRED`
* `APPROVED`

Status tersebut melekat pada rekap lokasi, bukan pada setiap transaksi individual.

---

## BR-035 — Status DRAFT

Rekap lokasi berstatus `DRAFT` selama proses pendataan masih berlangsung dan belum dikirim kepada Admin Desa.

---

## BR-036 — Pengiriman Rekap

Petugas Lokasi hanya dapat mengirim rekap lokasi yang memiliki data zakat valid.

Rekap kosong tidak dapat dikirim.

---

## BR-037 — SUBMITTED

Setelah Petugas Lokasi mengirim rekap, status berubah menjadi `SUBMITTED`.

Sistem harus mencatat:

* siapa yang mengirim;
* kapan rekap dikirim.

---

## BR-038 — Pemeriksaan oleh Admin

Hanya pengguna dengan kewenangan administratif yang dapat memeriksa rekap berstatus `SUBMITTED`.

---

## BR-039 — Permintaan Revisi

Admin Desa dapat mengubah status rekap dari:

`SUBMITTED`

menjadi:

`REVISION_REQUIRED`

apabila ditemukan data yang perlu diperbaiki.

---

## BR-040 — Alasan Revisi Wajib

Setiap permintaan revisi harus disertai alasan atau catatan perbaikan.

---

## BR-041 — Pengiriman Ulang Setelah Revisi

Setelah Petugas Lokasi menyelesaikan perbaikan, rekap harus dikirim ulang untuk diperiksa kembali.

Alur konseptual:

`REVISION_REQUIRED`

→ perbaikan

→ `SUBMITTED`

---

## BR-042 — Persetujuan Rekap

Admin Desa dapat mengubah rekap berstatus `SUBMITTED` menjadi `APPROVED` setelah proses pemeriksaan selesai.

---

## BR-043 — Rekap Resmi Desa

Hanya rekap lokasi berstatus `APPROVED` yang digunakan sebagai bagian dari rekap resmi desa.

---

# 9. Rekapitulasi Desa

## BR-044 — Rekap Desa Berdasarkan Periode

Rekap desa harus dibuat berdasarkan satu periode zakat tertentu.

Data dari periode berbeda tidak boleh digabung.

---

## BR-045 — Sumber Rekap Desa

Rekap resmi desa merupakan agregasi dari seluruh rekap lokasi yang berstatus `APPROVED`.

---

## BR-046 — Total Desa

Total tingkat desa meliputi minimal:

* jumlah lokasi yang masuk rekap;
* jumlah KK;
* jumlah jiwa;
* total beras;
* total uang.

---

## BR-047 — Rekap Berdasarkan Jenis Lokasi

Sistem dapat mengelompokkan rekap berdasarkan jenis lokasi:

* Masjid
* Musholla
* SD
* TK

tanpa mengubah data transaksi asal.

---

# 10. Laporan

## BR-048 — Sumber Data Laporan

Laporan resmi harus menggunakan data yang memenuhi aturan rekap yang berlaku.

---

## BR-049 — Laporan Per Lokasi

Laporan per lokasi harus merepresentasikan data dari satu lokasi dan periode yang dipilih.

---

## BR-050 — Laporan Desa

Laporan desa harus menggunakan data rekap resmi desa pada periode yang dipilih.

---

## BR-051 — Export Tidak Mengubah Data

Proses menghasilkan PDF atau Excel tidak boleh mengubah data transaksi atau rekap utama.

---

# 11. Audit Trail

## BR-052 — Aktivitas Penting Harus Dicatat

Aktivitas penting harus dapat dicatat dalam audit log.

Aktivitas dapat mencakup:

* pembuatan data;
* perubahan data;
* pembatalan data;
* perubahan konfigurasi;
* pengiriman rekap;
* permintaan revisi;
* persetujuan rekap;
* perubahan pengguna;
* perubahan hak akses.

---

## BR-053 — Histori Perubahan Data

Perubahan penting terhadap data zakat harus dapat dilacak.

Informasi audit minimal dapat mencakup:

* pengguna;
* waktu;
* jenis tindakan;
* data atau objek yang terkena perubahan.

Untuk perubahan tertentu, sistem dapat menyimpan nilai sebelum dan sesudah perubahan.

---

## BR-054 — Audit Log Tidak Dapat Diubah Petugas

Petugas Lokasi tidak dapat mengubah atau menghapus audit log.

---

## BR-055 — Pembatalan Tetap Terlacak

Transaksi yang dibatalkan harus tetap memiliki histori yang dapat dilacak melalui sistem.

---

# 12. Data Historis

## BR-056 — Data Periode Lama Tetap Disimpan

Data zakat dari periode sebelumnya harus tetap tersedia sebagai histori sesuai kebijakan penyimpanan data.

---

## BR-057 — Periode Baru Tidak Mengubah Data Lama

Pembuatan periode baru tidak boleh mengubah:

* transaksi periode sebelumnya;
* rekap sebelumnya;
* laporan sebelumnya;
* ketentuan zakat sebelumnya.

---

## BR-058 — Referensi Historis Tetap Dipertahankan

Penonaktifan pengguna atau lokasi tidak boleh memutus hubungan dengan data historis yang sebelumnya dibuat.

---

# 13. Validasi dan Integritas Data

## BR-059 — Validasi Dilakukan di Server

Aturan bisnis kritis harus divalidasi pada backend/server.

Validasi pada aplikasi mobile atau web hanya berfungsi sebagai lapisan tambahan untuk meningkatkan pengalaman pengguna.

---

## BR-060 — Transaksi Harus Konsisten

Sistem tidak boleh menyimpan transaksi zakat yang melanggar aturan wajib seperti:

* jumlah jiwa tidak valid;
* tidak memiliki lokasi;
* tidak memiliki periode;
* tidak memiliki bentuk pembayaran;
* nilai pembayaran negatif.

---

## BR-061 — Rekap Harus Dihitung dari Data Sumber

Nilai rekap harus berasal dari perhitungan data transaksi yang valid.

Sistem tidak boleh bergantung hanya pada angka total yang dimasukkan secara manual oleh pengguna.

---
## BR-062 — Satu Pencatatan Keluarga per Periode

Dalam satu periode zakat, sistem harus menggunakan satu pencatatan utama untuk satu keluarga pada konteks lokasi pembayaran yang sama.

Apabila keluarga yang sama melakukan pembayaran zakat lebih dari satu kali, sistem tidak membuat pencatatan keluarga baru.

Pembayaran berikutnya harus ditambahkan sebagai riwayat pembayaran pada pencatatan keluarga yang sudah ada.

Contoh:

Keluarga Ahmad telah melakukan pembayaran pertama untuk 2 jiwa menggunakan beras.

Pada hari berikutnya, keluarga Ahmad melakukan pembayaran tambahan untuk 1 jiwa menggunakan uang.

Sistem harus mempertahankan kedua pembayaran tersebut dalam pencatatan keluarga Ahmad yang sama.

---

## BR-063 — Riwayat Pembayaran Keluarga

Satu pencatatan keluarga dapat memiliki satu atau lebih riwayat pembayaran zakat.

Setiap pembayaran harus disimpan secara terpisah agar histori pembayaran tetap dapat dilacak.

Informasi pembayaran minimal dapat mencakup:

* tanggal pembayaran;
* jumlah jiwa;
* jumlah jiwa dengan pembayaran beras;
* jumlah beras;
* jumlah jiwa dengan pembayaran uang;
* nominal uang;
* petugas yang mencatat.

Pembayaran baru tidak boleh menimpa pembayaran sebelumnya.

---

## BR-064 — Pembayaran Tambahan

Apabila keluarga melakukan pembayaran tambahan setelah pembayaran sebelumnya telah dicatat, Petugas Lokasi harus dapat menambahkan pembayaran baru pada pencatatan keluarga tersebut.

Total zakat keluarga dihitung dari seluruh pembayaran yang valid pada pencatatan keluarga tersebut.

Contoh:

Pembayaran pertama:

* 2 jiwa
* 5 kg beras

Pembayaran kedua:

* 1 jiwa
* pembayaran uang

Maka total keluarga menjadi:

* 3 jiwa;
* 5 kg beras;
* ditambah nominal uang dari pembayaran kedua.

---

## BR-065 — Pembayaran Campuran

Satu pembayaran zakat dapat menggunakan lebih dari satu bentuk pembayaran.

Bentuk pembayaran V1 adalah:

* beras;
* uang.

Pembayaran campuran diperbolehkan apabila sebagian jiwa membayar menggunakan beras dan sebagian lainnya menggunakan uang.

---

## BR-066 — Pembagian Jiwa Berdasarkan Bentuk Pembayaran

Apabila satu pembayaran menggunakan beras dan uang sekaligus, sistem harus dapat mencatat jumlah jiwa untuk masing-masing bentuk pembayaran.

Contoh:

Total jiwa yang dibayarkan: 5 jiwa.

Pembagian:

* 3 jiwa menggunakan beras;
* 2 jiwa menggunakan uang.

Sistem harus mempertahankan informasi pembagian tersebut untuk kebutuhan rekapitulasi.

---

## BR-067 — Konsistensi Jumlah Jiwa Pembayaran

Jumlah jiwa berdasarkan seluruh bentuk pembayaran dalam satu pembayaran harus sama dengan jumlah jiwa pada pembayaran tersebut.

Contoh valid:

* Jiwa beras: 3
* Jiwa uang: 2
* Total jiwa pembayaran: 5

Contoh tidak valid:

* Jiwa beras: 3
* Jiwa uang: 2
* Total jiwa pembayaran: 6

Sistem harus menolak data yang tidak konsisten.

---

## BR-068 — Rekap Keluarga

Total zakat pada tingkat keluarga harus dihitung dari seluruh riwayat pembayaran yang valid.

Rekap keluarga minimal mencakup:

* total jiwa;
* total jiwa pembayaran beras;
* total beras;
* total jiwa pembayaran uang;
* total uang.

Transaksi atau pembayaran yang telah dibatalkan tidak boleh dimasukkan dalam perhitungan.

---

## BR-069 — Deteksi Pencatatan Keluarga yang Sudah Ada

Ketika Petugas Lokasi menyimpan pembayaran zakat, sistem harus melakukan pemeriksaan untuk mengetahui apakah keluarga tersebut telah memiliki pencatatan pada periode dan konteks lokasi yang sama.

Petugas tidak diwajibkan melakukan pencarian keluarga secara manual sebelum mengisi formulir pembayaran.

Jika sistem tidak menemukan keluarga yang sesuai, sistem dapat membuat pencatatan keluarga baru dan menghubungkan pembayaran tersebut dengan pencatatan baru.

Jika sistem menemukan satu atau lebih keluarga yang berpotensi sama, sistem harus menampilkan kandidat kepada Petugas Lokasi untuk dikonfirmasi.

Sistem tidak boleh menggabungkan pembayaran dengan pencatatan keluarga yang sudah ada secara otomatis apabila identitas keluarga belum dapat dipastikan.

Setelah Petugas Lokasi mengonfirmasi keluarga yang sesuai, pembayaran baru ditambahkan sebagai riwayat pembayaran keluarga tersebut tanpa menimpa pembayaran sebelumnya.

Pemeriksaan keluarga tidak boleh hanya menggunakan nama kepala keluarga sebagai satu-satunya dasar identifikasi karena terdapat kemungkinan beberapa keluarga memiliki nama kepala keluarga yang sama.

Kriteria pencocokan keluarga secara detail akan ditentukan pada Database Design berdasarkan data identifikasi yang tersedia dalam proses operasional.

---
# 14. Aturan yang Belum Dikunci

Beberapa aturan sengaja belum ditetapkan secara final karena membutuhkan validasi proses lapangan.

## BR-PENDING-003 — Admin Override

Belum ditentukan secara final apakah Admin Desa dapat membuat atau mengubah transaksi zakat secara langsung.

Kewenangan ini akan ditentukan pada Authorization Design.

---

## BR-PENDING-004 — Perubahan Setelah Approval

Belum ditentukan mekanisme administratif apabila ditemukan kesalahan setelah rekap berstatus `APPROVED`.

Kemungkinan mekanisme antara lain:

* reopen;
* administrative correction;
* approval ulang.

Keputusan akan dibuat apabila kebutuhan tersebut terbukti diperlukan.

---

# 15. Ringkasan Workflow Utama

Alur status rekap lokasi:

`DRAFT`

→ Petugas mengirim rekap

→ `SUBMITTED`

Kemudian:

`SUBMITTED`

→ Admin menerima data

→ `APPROVED`

atau:

`SUBMITTED`

→ Admin menemukan masalah

→ `REVISION_REQUIRED`

→ Petugas memperbaiki data

→ `SUBMITTED`

→ Admin memeriksa kembali

→ `APPROVED`

---

# 16. Change Management

Perubahan terhadap Business Rule harus dicatat pada `change-log.md`.

Setiap perubahan harus dianalisis dampaknya terhadap:

* Functional Requirements;
* Use Case;
* Workflow;
* Database;
* API;
* UI;
* Security;
* Testing;
* Reporting.

Business Rule yang berubah tidak boleh hanya diubah pada implementasi kode tanpa memperbarui dokumentasi yang terkait.

---

# 17. Kesimpulan

Business Rules V1 mengatur empat area utama sistem:

1. validitas pendataan zakat fitrah;
2. kewenangan pengguna dan lokasi;
3. workflow pengiriman dan persetujuan rekap;
4. integritas, histori, dan audit data.

Status workflow diterapkan pada tingkat rekap lokasi, sedangkan transaksi individual tetap menjadi sumber data utama untuk perhitungan rekap.

Hanya rekap lokasi yang telah berstatus `APPROVED` yang digunakan dalam rekap resmi desa.

Dokumen ini menjadi dasar untuk tahap berikutnya yaitu Workflow Design.
