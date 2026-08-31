# System Design

## 1. Tujuan Dokumen

Dokumen ini mendefinisikan desain sistem tingkat tinggi untuk Sistem Pendataan dan Rekapitulasi Zakat Fitrah Desa.

System Design menjelaskan:

* batas sistem;
* jenis aplikasi yang digunakan;
* pengguna sistem;
* komponen utama;
* tanggung jawab setiap komponen;
* hubungan antarbagian sistem;
* aliran data;
* pengelolaan autentikasi dan otorisasi;
* mekanisme pendataan;
* rekapitulasi;
* workflow approval;
* reporting;
* audit;
* penanganan kesalahan;
* kebutuhan penyimpanan data;
* kebutuhan integrasi antarbagian sistem.

Dokumen ini menjadi penghubung antara:

Requirement → Use Case → Business Rules → Workflow → Architecture Design → Database Design → API Design → Implementation.

---

# 2. System Context

Sistem dirancang untuk mendigitalisasi proses pendataan dan rekapitulasi zakat fitrah pada tingkat desa.

Sistem melibatkan tiga pengguna utama:

1. Petugas Lokasi
2. Admin Desa
3. DKM

Warga atau muzakki tidak menjadi pengguna langsung sistem pada V1.

Alur interaksi tingkat tinggi:

```text
                    WARGA / MUZAKKI
                           │
                           │ Menunaikan zakat
                           ▼
                    PETUGAS LOKASI
                           │
                           │ Input pembayaran
                           ▼
                  ┌──────────────────┐
                  │                  │
                  │  SISTEM ZAKAT    │
                  │                  │
                  └────────┬─────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
        ADMIN DESA        DKM        DATABASE
              │            │
              ▼            ▼
         Rekap Desa      Laporan
```

---

# 3. System Boundary

## 3.1 Termasuk Dalam Sistem

Sistem V1 menangani:

* autentikasi pengguna;
* otorisasi berdasarkan role;
* pengelolaan pengguna;
* pengelolaan lokasi;
* pengelolaan periode zakat;
* konfigurasi ketentuan zakat;
* pencatatan keluarga;
* pencatatan pembayaran;
* deteksi keluarga yang telah tercatat;
* pembayaran tambahan;
* pembayaran beras;
* pembayaran uang;
* pembayaran campuran;
* perubahan pembayaran;
* pembatalan pembayaran;
* rekapitulasi lokasi;
* workflow submission;
* workflow revisi;
* approval;
* rekapitulasi desa;
* dashboard;
* laporan;
* export laporan;
* audit log.

---

## 3.2 Di Luar Sistem V1

Sistem V1 belum menangani:

* zakat mal;
* perhitungan nisab;
* haul;
* distribusi zakat;
* pengelolaan mustahik;
* pembayaran melalui bank;
* payment gateway;
* e-wallet;
* pembayaran online oleh warga;
* aplikasi warga;
* donasi umum.

---

# 4. High-Level System Design

Secara logis sistem dibagi menjadi:

                  ┌────────────────────────┐
                  │   MOBILE APPLICATION   │
                  │                        │
                  │ Petugas Lokasi         │
                  │ Admin Desa             │
                  │ DKM                    │
                  └───────────┬────────────┘
                              │
                            HTTPS
                              │
                              ▼
                     ┌─────────────────┐
                     │   BACKEND API   │
                     │                 │
                     │ Authentication  │
                     │ Authorization   │
                     │ Business Logic  │
                     │ Workflow        │
                     │ Reporting       │
                     │ Audit           │
                     └────────┬────────┘
                              │
                              ▼
                         ┌──────────┐
                         │ DATABASE │
                         └──────────┘

---

# 5. Client Applications

Sistem V1 menggunakan satu Mobile Application dengan interface dan fitur yang disesuaikan berdasarkan role pengguna. Petugas Lokasi, Admin Desa, dan DKM menggunakan aplikasi yang sama, tetapi backend membatasi fitur dan data berdasarkan role serta scope pengguna.

## 5.1 Mobile Application

Mobile Application terutama digunakan oleh Petugas Lokasi.

Tujuannya adalah menyediakan proses input yang sederhana dan cepat pada saat warga melakukan pembayaran.

Fitur utama mobile:

* login;
* dashboard lokasi;
* tambah pembayaran;
* melihat data keluarga;
* melihat pembayaran;
* mengubah pembayaran;
* membatalkan pembayaran;
* melihat rekap lokasi;
* melihat status rekap;
* mengirim rekap;
* melihat alasan revisi;
* memperbaiki data;
* mengirim ulang rekap.

Desain mobile harus memprioritaskan usability karena Petugas Lokasi dapat memiliki tingkat kemampuan teknologi yang berbeda.


* Admin Desa;
* DKM.

### Admin Desa

* dashboard desa;
* user management;
* location management;
* penugasan Petugas;
* konfigurasi periode;
* konfigurasi ketentuan zakat;
* melihat seluruh lokasi;
* melihat data zakat;
* memeriksa rekap;
* meminta revisi;
* menyetujui rekap;
* melihat status pengumpulan;
* melihat rekap desa;
* generate laporan;
* export laporan;
* melihat audit log.

### DKM

* dashboard;
* rekap lokasi;
* rekap desa;
* laporan;
* export laporan.

DKM tidak memiliki kewenangan operasional terhadap transaksi.

---

# 6. Backend System

Backend menjadi pusat logika dan sumber kebenaran utama sistem.

Backend bertanggung jawab untuk:

* autentikasi;
* otorisasi;
* business validation;
* business rules;
* pengelolaan data;
* workflow;
* aggregasi;
* audit;
* reporting;
* akses database.

Client tidak boleh menentukan sendiri apakah sebuah operasi diperbolehkan.

Contoh:

```text
Mobile menampilkan tombol Edit
          │
          ▼
Petugas klik Edit
          │
          ▼
Backend tetap memeriksa:
- role
- lokasi
- periode
- status rekap
          │
          ▼
Baru operasi diperbolehkan / ditolak
```

Backend merupakan authoritative source untuk aturan sistem.

---

# 7. Logical Backend Modules

Pada tingkat System Design, backend dapat dibagi menjadi beberapa logical module.

## 7.1 Authentication Module

Bertanggung jawab untuk:

* login;
* logout;
* validasi identitas pengguna;
* pengelolaan sesi/token;
* pemeriksaan status akun.

---

## 7.2 User Management Module

Bertanggung jawab untuk:

* data pengguna;
* role;
* status akun;
* penugasan Petugas.

---

## 7.3 Location Module

Bertanggung jawab untuk:

* Masjid;
* Musholla;
* SD;
* TK;
* status lokasi;
* hubungan Petugas dengan lokasi.

---

## 7.4 Zakat Period Module

Bertanggung jawab untuk:

* periode zakat;
* tanggal mulai;
* tanggal selesai;
* status periode;
* konfigurasi zakat berdasarkan periode.

---

## 7.5 Family Module

Bertanggung jawab untuk pencatatan keluarga.

Konsep keluarga digunakan agar satu keluarga tidak dihitung berulang kali hanya karena melakukan beberapa pembayaran.

Family Module bertanggung jawab untuk:

* membuat pencatatan keluarga;
* menyimpan informasi identifikasi;
* mencari kandidat keluarga;
* menghubungkan keluarga dengan lokasi dan periode;
* memberikan data kepada proses pembayaran.

---

## 7.6 Family Matching Module

Family Matching bertanggung jawab membantu mendeteksi pencatatan keluarga yang mungkin sudah tersedia.

Workflow:

```text
Input keluarga
      │
      ▼
Normalize data
      │
      ▼
Cari kandidat
      │
   ┌──┴────┐
Tidak      Ada
 │          │
 ▼          ▼
Create    Return kandidat
Family         │
               ▼
        Petugas konfirmasi
```

Sistem tidak boleh melakukan merge secara otomatis apabila hasil pencocokan belum cukup pasti.

---

## 7.7 Payment Module

Payment Module bertanggung jawab terhadap pembayaran zakat.

Satu keluarga dapat memiliki:

```text
1 Family
   │
   ├── Payment 1
   ├── Payment 2
   ├── Payment 3
   └── ...
```

Payment Module menangani:

* pembayaran baru;
* pembayaran tambahan;
* jiwa pembayaran;
* jiwa beras;
* berat beras;
* jiwa uang;
* nominal uang;
* tanggal pembayaran;
* perubahan;
* pembatalan.

---

## 7.8 Recap Module

Recap Module bertanggung jawab menghitung rekap otomatis.

Tingkat rekap:

```text
PAYMENTS
    ↓
FAMILY TOTAL
    ↓
LOCATION RECAP
    ↓
VILLAGE RECAP
```

Rekap tidak boleh bergantung pada total yang diketik manual.

---

## 7.9 Workflow Module

Workflow Module bertanggung jawab terhadap status rekap.

State:

```text
DRAFT
  ↓
SUBMITTED
  ↓
 ┌──────────────┐
 │              │
 ▼              ▼
REVISION      APPROVED
REQUIRED
 │
 ▼
SUBMITTED
```

Workflow Module memvalidasi:

* actor;
* current state;
* requested action;
* next state.

---

## 7.10 Reporting Module

Reporting Module bertanggung jawab:

* laporan lokasi;
* laporan desa;
* preview laporan;
* PDF;
* Excel.

Reporting harus menggunakan sumber data yang sama dengan dashboard dan rekap sehingga tidak terdapat perbedaan angka.

---

## 7.11 Audit Module

Audit Module bertanggung jawab mencatat aktivitas penting.

Contoh:

```text
Actor:
Petugas A

Action:
UPDATE_PAYMENT

Target:
Payment 123

Timestamp:
...

Before:
2 jiwa

After:
3 jiwa
```

---

# 8. Logical Data Model

Pada tahap System Design, data belum didefinisikan sampai tingkat kolom database.

Namun hubungan konsep utama dapat digambarkan sebagai berikut:

```text
USER
 │
 ├──── assigned to ──── LOCATION
 │
 │
 │                        │
 │                        │
 │                        ▼
 │                    FAMILY RECORD
 │                        │
 │                        │ 1
 │                        │
 │                        │ N
 │                        ▼
 │                     PAYMENT
 │
 │
 └──── performs ────── AUDIT LOG


PERIOD
 │
 ├──── FAMILY RECORD
 │
 ├──── PAYMENT
 │
 └──── LOCATION RECAP


LOCATION
    │
    └── LOCATION RECAP
            │
            ▼
       VILLAGE RECAP
```

Detail entity dan atribut akan ditentukan pada `13-database-design.md`.

---

# 9. Family and Payment Relationship

Salah satu keputusan desain utama adalah memisahkan:

* keluarga;
* pembayaran.

Tidak menggunakan model:

```text
Ahmad | 3 jiwa | 5 kg | Rp100.000
```

sebagai satu record yang terus ditimpa.

Sebaliknya:

```text
Family Ahmad
│
├── Payment #1
│   ├── 2 jiwa
│   ├── 5 kg
│   └── Rp0
│
└── Payment #2
    ├── 1 jiwa
    ├── 0 kg
    └── Rp...
```

Keuntungan desain:

* histori pembayaran tersimpan;
* pembayaran tambahan dapat dicatat;
* audit lebih jelas;
* jumlah KK tidak terduplikasi;
* rekap lebih akurat.

---

# 10. Payment Validation Design

Backend harus memvalidasi pembayaran sebelum penyimpanan.

Validasi dasar:

```text
totalJiwa >= 1

jiwaBeras >= 0

jiwaUang >= 0

beras >= 0

uang >= 0
```

dan:

```text
jiwaBeras + jiwaUang = totalJiwa
```

Minimal satu metode pembayaran harus digunakan.

Contoh valid:

```text
Total Jiwa = 4

Jiwa Beras = 4
Beras = 10 kg

Jiwa Uang = 0
Uang = 0
```

Contoh valid:

```text
Total Jiwa = 4

Jiwa Beras = 2
Beras = 5 kg

Jiwa Uang = 2
Uang = Rp...
```

Contoh tidak valid:

```text
Total Jiwa = 4

Jiwa Beras = 2
Jiwa Uang = 1
```

karena:

```text
2 + 1 ≠ 4
```

---

# 11. Family Detection Design

Family detection dilakukan setelah form pembayaran valid.

Sistem dapat menggunakan beberapa data untuk mencari kandidat keluarga.

Contoh informasi kandidat:

* nama kepala keluarga;
* wilayah/RT/RW;
* alamat;
* informasi identifikasi lain yang tersedia.

Nama kepala keluarga tidak boleh menjadi satu-satunya identifier teknis.

Contoh:

```text
Input:

Nama : Ahmad
RT   : 02
RW   : 01
```

Sistem mencari kandidat:

```text
Ahmad - RT 02/RW 01
Ahmad - RT 03/RW 01
```

Petugas kemudian memilih kandidat yang benar.

Algoritma pencocokan detail ditentukan kemudian.

Pada V1 tidak perlu menggunakan AI atau machine learning untuk family matching.

---

# 12. Recap Calculation Design

## 12.1 Family Level

Satu keluarga dapat memiliki beberapa pembayaran.

Contoh:

```text
Family A

Payment #1 = 2 jiwa
Payment #2 = 1 jiwa
```

Maka:

```text
Family count = 1 KK
Total jiwa = 3
```

---

## 12.2 Location Level

Location Recap dihitung dari seluruh keluarga dan pembayaran valid pada lokasi dan periode yang sama.

Minimal:

```text
Total KK
Total Jiwa
Jiwa Beras
Total Beras
Jiwa Uang
Total Uang
```

---

## 12.3 Village Level

Village Recap merupakan agregasi Location Recap yang:

```text
status = APPROVED
```

Contoh:

```text
Location A → APPROVED ──┐
Location B → APPROVED ──┤
Location C → SUBMITTED X│
Location D → APPROVED ──┘
                        │
                        ▼
                  Village Recap
```

Location C tidak masuk dalam rekap resmi.

---

# 13. Recap Calculation Strategy

Secara logis terdapat dua kemungkinan implementasi:

### Strategy A — Calculate On Demand

Setiap kali rekap diminta:

```text
Database
   ↓
Hitung ulang
   ↓
Tampilkan
```

### Strategy B — Stored Summary

Sistem menyimpan hasil agregasi:

```text
Payment berubah
      ↓
Update summary
      ↓
Simpan recap
```

Keputusan teknis final antara calculate-on-demand, stored summary, atau kombinasi keduanya akan dibuat pada Database dan Architecture Design.

Yang wajib:

> hasil rekap harus dapat ditelusuri kembali ke data pembayaran sumber.

---

# 14. Recap State Design

Location Recap memiliki state:

```text
DRAFT
SUBMITTED
REVISION_REQUIRED
APPROVED
```

State transition:

| Current           | Action           | Actor   | Next              |
| ----------------- | ---------------- | ------- | ----------------- |
| DRAFT             | Submit           | Petugas | SUBMITTED         |
| SUBMITTED         | Request Revision | Admin   | REVISION_REQUIRED |
| SUBMITTED         | Approve          | Admin   | APPROVED          |
| REVISION_REQUIRED | Resubmit         | Petugas | SUBMITTED         |

Tidak diperbolehkan:

```text
DRAFT → APPROVED
```

atau:

```text
Petugas:
SUBMITTED → APPROVED
```

Backend harus memvalidasi state transition.

---

# 15. Data Locking Design

Editability mengikuti status rekap.

## DRAFT

```text
Add     ✅
Edit    ✅
Cancel  ✅
```

## SUBMITTED

```text
Add     ❌
Edit    ❌
Cancel  ❌
```

## REVISION_REQUIRED

```text
Add     ✅
Edit    ✅
Cancel  ✅
```

## APPROVED

```text
Add     ❌
Edit    ❌
Cancel  ❌
```

Pengecekan harus dilakukan oleh backend.

Tidak cukup hanya menyembunyikan tombol pada UI.

---

# 16. Authentication Design

Sistem membutuhkan authentication layer.

Konsep:

```text
User
 ↓
Login
 ↓
Credentials Validation
 ↓
Authentication Success
 ↓
Authentication Credential / Session
 ↓
Access API
```

Mekanisme teknis seperti JWT, session, refresh token, atau pendekatan lain akan ditentukan pada Architecture dan Security Design.

---

# 17. Authorization Design

Authorization menggunakan minimal dua dimensi:

```text
ROLE
+
RESOURCE SCOPE
```

Contoh:

Petugas Lokasi:

```text
Role:
PETUGAS

Scope:
LOCATION_A
```

Maka:

```text
GET Location A Data → Allowed

GET Location B Data → Forbidden
```

Admin Desa:

```text
Scope:
Village
```

DKM:

```text
Scope:
Read-only reporting
```

---

# 18. Authorization Flow

Setiap protected request mengikuti pola:

```text
Request
   ↓
Authentication Check
   ↓
User valid?
   │
   ├── No → 401
   │
   └── Yes
        ↓
Authorization Check
        ↓
Allowed?
   │
   ├── No → 403
   │
   └── Yes
        ↓
Business Validation
        ↓
Execute Operation
```

---

# 19. Audit Design

Audit log harus dipisahkan secara konseptual dari data bisnis utama.

Audit menjawab:

```text
WHO?
WHAT?
WHEN?
ON WHICH DATA?
```

Untuk perubahan tertentu:

```text
BEFORE?
AFTER?
```

Contoh aktivitas yang dicatat:

* login penting jika diperlukan;
* create family;
* create payment;
* update payment;
* cancel payment;
* submit recap;
* request revision;
* resubmit;
* approve;
* change role;
* deactivate account;
* change configuration.

---

# 20. Reporting Design

Reporting menggunakan data sistem sebagai source of truth.

Alur:

```text
User memilih laporan
        ↓
Backend validasi permission
        ↓
Backend mengambil data
        ↓
Reporting service
        ↓
Generate
        ↓
PDF / Excel
```

Format laporan V1:

* rekap lokasi;
* rekap desa.

---

# 21. Dashboard Design

## 21.1 Petugas Dashboard

Minimal menampilkan:

* nama lokasi;
* periode aktif;
* jumlah KK;
* jumlah jiwa;
* jiwa beras;
* total beras;
* jiwa uang;
* total uang;
* status rekap;
* alasan revisi jika ada.

---

## 21.2 Admin Dashboard

Minimal menampilkan:

* periode;
* jumlah lokasi;
* jumlah lokasi `DRAFT`;
* jumlah lokasi `SUBMITTED`;
* jumlah lokasi `REVISION_REQUIRED`;
* jumlah lokasi `APPROVED`;
* total KK approved;
* total jiwa approved;
* total beras approved;
* total uang approved.

---

## 21.3 DKM Dashboard

Minimal menampilkan informasi read-only:

* rekap desa;
* rekap lokasi;
* statistik utama;
* laporan.

---

# 22. Search and Filter Design

Sistem harus mendukung pencarian dan filtering terhadap data yang relevan.

Contoh:

### Petugas

* nama kepala keluarga;
* tanggal pembayaran.

### Admin

* lokasi;
* jenis lokasi;
* periode;
* status;
* nama kepala keluarga;
* tanggal.

Search dan filter harus tetap mengikuti authorization.

Petugas tidak boleh memperoleh data lokasi lain hanya karena menggunakan endpoint pencarian.

---

# 23. Error Handling Design

Sistem membedakan:

## Validation Error

Contoh:

```text
Jumlah jiwa harus minimal 1.
```

## Authentication Error

```text
Sesi tidak valid atau telah berakhir.
```

## Authorization Error

```text
Anda tidak memiliki akses terhadap data ini.
```

## Workflow Conflict

```text
Rekap sudah dikirim dan tidak dapat diubah.
```

## Not Found

```text
Data tidak ditemukan.
```

## Internal Error

Pengguna menerima pesan yang aman dan mudah dipahami.

Detail teknis dicatat pada server log.

---

# 24. Error Response Principle

Backend tidak boleh mengirim informasi sensitif seperti:

* stack trace;
* database credentials;
* SQL internal;
* secret;
* path server internal.

Pesan teknis hanya tersedia di logging internal.

---

# 25. Transactional Consistency

Operasi yang melibatkan beberapa perubahan data harus mempertimbangkan atomicity.

Contoh:

```text
Create Family
+
Create Payment
+
Create Audit
```

Jika proses utama gagal di tengah jalan, sistem tidak boleh meninggalkan kondisi seperti:

```text
Family berhasil dibuat
tetapi
Payment gagal
```

tanpa penanganan yang sesuai.

Database transaction akan dipertimbangkan untuk operasi kritis.

---

# 26. Duplicate Request Protection

Sistem harus mempertimbangkan situasi:

```text
Petugas klik SIMPAN
        ↓
Internet lambat
        ↓
Petugas klik SIMPAN lagi
```

Sistem harus meminimalkan risiko satu pembayaran tersimpan dua kali akibat request yang sama dikirim berulang.

Strategi teknis seperti:

* idempotency key;
* request identifier;
* client-generated identifier;

akan ditentukan pada API Design.

---

# 27. Concurrency Design

Backend harus memvalidasi kondisi terakhir data sebelum melakukan update.

Contoh:

```text
Petugas membuka form edit ketika status DRAFT

sementara itu

rekap berubah menjadi SUBMITTED
```

Ketika Petugas menekan Simpan:

```text
Backend cek status terbaru
        ↓
SUBMITTED
        ↓
Update ditolak
```

Client state tidak boleh dianggap sebagai source of truth.

---

# 28. Data Integrity

Sistem harus menjaga hubungan antar data.

Contoh:

```text
Payment
harus memiliki
Family

Family
harus memiliki
Location dan Period

Location Recap
harus memiliki
Location dan Period
```

Data orphan harus dicegah.

Detail foreign key akan ditentukan pada Database Design.

---

# 29. Historical Data Design

Sistem menggunakan pendekatan historis.

Periode lama tetap tersedia.

Contoh:

```text
2026
├── Families
├── Payments
├── Recaps
└── Reports

2027
├── Families
├── Payments
├── Recaps
└── Reports
```

Data 2027 tidak boleh mengubah data 2026.

---

# 30. Deactivation Instead of Destructive Deletion

Untuk beberapa master data:

```text
USER
LOCATION
```

lebih baik menggunakan status:

```text
ACTIVE
INACTIVE
```

daripada langsung menghapus data yang sudah memiliki histori.

Hal ini menjaga referential integrity dan auditability.

---

# 31. Communication Design

Client berkomunikasi dengan backend melalui API.

Konsep:

```text
Mobile
  │
  ├─────────────┐
  │             │
  ▼             │
                │
              API
                │
  ▲             │
  │             │
Web ────────────┘
                │
                ▼
            Database
```

Client tidak berkomunikasi langsung dengan database.

---

# 32. Data Flow — Input Pembayaran

```text
Mobile
  │
  │ Payment Form
  ▼
API
  │
  ├─ Authenticate
  ├─ Authorize
  ├─ Validate
  ├─ Match Family
  │
  ▼
Family / Payment Service
  │
  ▼
Database
  │
  ├─ Family
  ├─ Payment
  └─ Audit
  │
  ▼
Response
  │
  ▼
Mobile
```

---

# 33. Data Flow — Submit Rekap

```text
Mobile Petugas
      │
      ▼
Submit Request
      │
      ▼
Backend
      │
      ├─ Authentication
      ├─ Authorization
      ├─ Validate status
      ├─ Calculate recap
      └─ Change state
              │
              ▼
          SUBMITTED
              │
              ▼
           Database
```

---

# 34. Data Flow — Approval

```text
Admin Web
   │
   ▼
Approve
   │
   ▼
Backend
   │
   ├─ Authenticate Admin
   ├─ Check permission
   ├─ Check SUBMITTED
   ├─ Approve recap
   └─ Audit
         │
         ▼
      APPROVED
         │
         ▼
Village Recap
```

---

# 35. Data Flow — Reporting

```text
Admin / DKM
      │
      ▼
Select Report
      │
      ▼
Backend
      │
      ├─ Authorization
      ├─ Query approved data
      ├─ Aggregate
      └─ Generate document
              │
              ▼
          PDF / Excel
```

---

# 36. Logical Security Layers

Sistem minimal memiliki lapisan:

```text
HTTPS
  ↓
Authentication
  ↓
Authorization
  ↓
Input Validation
  ↓
Business Rules
  ↓
Database Constraints
  ↓
Audit
```

Security tidak boleh hanya diterapkan pada satu lapisan.

---

# 37. Logging

Logging teknis berbeda dengan Audit Log.

## Application Logging

Digunakan developer/operator untuk:

* error;
* warning;
* server behavior;
* debugging produksi.

## Audit Logging

Digunakan untuk menjawab aktivitas bisnis pengguna.

Contoh:

```text
Application Log:
Database timeout.

Audit Log:
Admin A approved Recap B.
```

Kedua konsep tidak boleh dianggap sama.

---

# 38. Backup and Recovery

Database harus memiliki mekanisme backup.

Tujuan:

* mengurangi risiko kehilangan data;
* memungkinkan recovery setelah kegagalan;
* menjaga data historis.

Detail:

* frekuensi backup;
* retention;
* lokasi backup;
* restore procedure;

akan ditentukan pada Deployment Design.

---

# 39. Availability Consideration

Sistem terutama kritis digunakan selama periode menjelang Idul Fitri.

Karena itu sistem harus dirancang agar:

* layanan backend tersedia;
* database dapat diakses;
* backup berjalan;
* error dapat dilacak;
* sistem tidak kehilangan pembayaran ketika terjadi kegagalan.

Target availability numerik akan ditentukan setelah infrastruktur deployment dipilih.

---

# 40. Performance Consideration

Operasi utama seperti:

* login;
* membuka data;
* menyimpan pembayaran;
* melihat rekap;
* membuka dashboard;

harus memberikan waktu respons yang layak pada kondisi normal.

Optimasi dapat dilakukan melalui:

* indexing database;
* pagination;
* query optimization;
* caching jika benar-benar diperlukan.

Caching tidak wajib pada V1 jika belum terdapat masalah performa.

---

# 41. Pagination

Daftar dengan jumlah data besar harus mendukung pagination.

Contoh:

* keluarga;
* pembayaran;
* audit log;
* daftar pengguna.

Sistem tidak disarankan mengambil seluruh data sekaligus apabila jumlah data dapat terus bertambah.

---

# 42. Scalability

V1 ditujukan untuk penggunaan tingkat desa.

Namun desain harus tetap memungkinkan:

```text
1 lokasi
→ 10 lokasi
→ puluhan lokasi
```

dan:

```text
1 periode
→ banyak periode tahunan
```

tanpa perubahan fundamental pada model sistem.

Sistem tidak perlu menggunakan microservices hanya untuk memenuhi scalability V1.

---

# 43. Extensibility

Struktur sistem sebaiknya memungkinkan penambahan modul di masa depan.

Contoh:

```text
CURRENT
Zakat Fitrah

FUTURE
├── Zakat Mal
├── Mustahik
├── Distribution
└── Other Reporting
```

Penambahan tersebut tidak boleh memaksa perubahan besar terhadap seluruh modul yang sudah stabil.

---

# 44. Technology Boundary

Pada tahap System Design, konsep berikut belum dianggap keputusan final:

* framework backend;
* framework web;
* database engine;
* authentication mechanism;
* deployment provider;
* cloud provider;
* container strategy;
* CI/CD platform.

Candidate technology dapat dibahas pada Architecture Design.

Dengan demikian System Design tetap berfokus pada kebutuhan sistem, bukan produk atau framework tertentu.

---

# 45. Current Technology Candidates

Berdasarkan kebutuhan dan konteks pengembangan, kandidat teknologi dapat berupa:

```text
Mobile:
Flutter

Web:
Web frontend framework

Backend:
REST API backend

Database:
Relational Database

Reporting:
Server-side PDF / Excel generation
```

Pemilihan final beserta alasannya akan dicatat pada:

`12-architecture-design.md`

dan jika diperlukan:

`docs/decisions/ADR-xxx.md`

---

# 46. Pending System Decisions

## SD-PENDING-001 — Admin Override

Belum diputuskan apakah Admin Desa dapat membuat, mengubah, atau membatalkan pembayaran secara langsung.

---

## SD-PENDING-002 — Correction After Approval

Belum diputuskan mekanisme koreksi apabila kesalahan ditemukan setelah rekap berstatus `APPROVED`.

---

## SD-PENDING-003 — Period Closing Rule

Belum diputuskan apakah periode dapat ditutup ketika masih terdapat rekap:

* `DRAFT`;
* `SUBMITTED`;
* `REVISION_REQUIRED`.

---

## SD-PENDING-004 — Family Identifier

Belum diputuskan kombinasi data terbaik untuk membantu mengidentifikasi keluarga secara konsisten.

Keputusan harus mempertimbangkan data yang benar-benar tersedia di lapangan.

---

## SD-PENDING-005 — Offline Capability

V1 belum menetapkan apakah aplikasi Petugas harus dapat melakukan input ketika tidak memiliki koneksi internet.

Kebutuhan ini harus diputuskan sebelum implementasi mobile karena dapat memengaruhi:

* local storage;
* synchronization;
* conflict handling;
* API design;
* data consistency.

---

# 47. System Design Principles

V1 mengikuti beberapa prinsip utama.

### Principle 1 — Server as Source of Truth

Backend menjadi sumber kebenaran terhadap status, permission, dan business rules.

### Principle 2 — Preserve History

Perubahan penting tidak boleh menghilangkan histori tanpa alasan.

### Principle 3 — Simple User Experience

Kompleksitas sistem sebisa mungkin ditangani oleh backend, bukan dibebankan kepada Petugas.

Contoh:

Petugas tidak perlu mencari keluarga terlebih dahulu.

Sistem membantu mendeteksinya.

### Principle 4 — Automatic Aggregation

Rekap dihitung dari data sumber.

### Principle 5 — Least Privilege

Pengguna hanya memperoleh akses sesuai kebutuhan tugasnya.

### Principle 6 — Modular Design

Domain utama dipisahkan secara logis agar mudah dikembangkan dan dipelihara.

### Principle 7 — Avoid Premature Complexity

Sistem tidak menggunakan teknologi kompleks tanpa kebutuhan nyata.

---

# 48. High-Level Component Summary

| Component        | Responsibility                      |
| ---------------- | ----------------------------------- |
| Mobile App       | Operasional Petugas Lokasi          |
| Web App          | Administrasi Desa dan reporting DKM |
| Backend API      | Business logic dan pusat akses data |
| Authentication   | Identitas pengguna                  |
| Authorization    | Role dan resource scope             |
| User Module      | Pengguna dan role                   |
| Location Module  | Lokasi penerimaan zakat             |
| Period Module    | Periode dan konfigurasi             |
| Family Module    | Pencatatan keluarga                 |
| Family Matching  | Deteksi keluarga existing           |
| Payment Module   | Riwayat pembayaran                  |
| Recap Module     | Agregasi lokasi/desa                |
| Workflow Module  | State transition rekap              |
| Reporting Module | PDF/Excel                           |
| Audit Module     | Histori aktivitas                   |
| Database         | Persistent data storage             |

---

# 49. System Design Overview

```text
                         USERS
          ┌──────────────┼───────────────┐
          │              │               │
          ▼              ▼               ▼
     Petugas          Admin Desa         DKM
          │              │               │
          ▼              └───────┬───────┘
    MOBILE APP                   │
          │                      ▼
          │                   WEB APP
          │                      │
          └──────────┬───────────┘
                     │
                   HTTPS
                     │
                     ▼
          ┌─────────────────────┐
          │     BACKEND API     │
          ├─────────────────────┤
          │ Authentication      │
          │ Authorization       │
          │ User                │
          │ Location            │
          │ Period              │
          │ Family              │
          │ Family Matching     │
          │ Payment             │
          │ Recap               │
          │ Workflow            │
          │ Reporting           │
          │ Audit               │
          └──────────┬──────────┘
                     │
                     ▼
              ┌─────────────┐
              │  DATABASE   │
              └─────────────┘
```

---

# 50. Conclusion

Sistem V1 dirancang sebagai sistem terpusat yang melayani Petugas Lokasi melalui aplikasi mobile dan Admin Desa/DKM melalui aplikasi web.

Backend menjadi pusat business logic, authentication, authorization, workflow, rekapitulasi, reporting, dan audit.

Data utama dipisahkan secara konseptual menjadi:

```text
Family
  ↓
Payments
  ↓
Location Recap
  ↓
Village Recap
```

Satu keluarga dapat memiliki lebih dari satu pembayaran.

Rekap lokasi dihitung otomatis dari pembayaran valid dan menggunakan workflow:

```text
DRAFT
→ SUBMITTED
→ REVISION_REQUIRED / APPROVED
```

Hanya rekap berstatus `APPROVED` yang masuk ke rekap resmi desa.

System Design ini menjadi dasar untuk tahap berikutnya yaitu Architecture Design.
