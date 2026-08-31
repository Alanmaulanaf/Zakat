# Architecture Design

## 1. Tujuan Dokumen

Dokumen ini mendefinisikan arsitektur teknis Sistem Pendataan dan Rekapitulasi Zakat Fitrah Desa V1.

Architecture Design menjelaskan:

* architectural style;
* teknologi utama;
* struktur aplikasi mobile;
* struktur backend;
* komunikasi client-server;
* database access;
* authentication;
* authorization;
* family identification;
* duplicate detection;
* payment processing;
* workflow rekap;
* reporting;
* audit;
* security architecture;
* error handling;
* concurrency;
* deployment;
* scalability;
* extensibility;
* strategi pengembangan offline di masa depan.

Dokumen ini menjadi dasar untuk:

* Database Design;
* API Design;
* Security Design;
* Testing Strategy;
* Deployment;
* Implementation.

---

# 2. Architecture Goals

Arsitektur V1 dirancang dengan tujuan:

1. Sederhana untuk dikembangkan dan dipelihara.
2. Cocok untuk single developer atau tim kecil.
3. Memiliki pemisahan tanggung jawab yang jelas.
4. Mudah diuji.
5. Mudah dikembangkan di masa depan.
6. Menjaga integritas dan histori data zakat.
7. Mendukung role dan scope authorization.
8. Menjaga pengalaman penggunaan Petugas tetap sederhana.
9. Tidak menggunakan kompleksitas teknologi yang belum diperlukan.
10. Tetap layak digunakan pada lingkungan produksi tingkat desa.

---

# 3. Architecture Decision Summary

| ID     | Decision                                                                   | Status |
| ------ | -------------------------------------------------------------------------- | ------ |
| AD-001 | Seluruh pengguna menggunakan satu Flutter Mobile Application               | Final  |
| AD-002 | Backend menggunakan Modular Monolith                                       | Final  |
| AD-003 | Komunikasi client-server menggunakan REST API melalui HTTPS                | Final  |
| AD-004 | Backend menjadi authoritative source untuk business rule dan authorization | Final  |
| AD-005 | Database menggunakan relational database                                   | Final  |
| AD-006 | Database utama menggunakan MySQL                                           | Final  |
| AD-007 | Backend menggunakan Node.js + TypeScript + Express.js                      | Final  |
| AD-008 | Mobile menggunakan Flutter + Dart                                          | Final  |
| AD-009 | Flutter menggunakan Riverpod                                               | Final  |
| AD-010 | Backend menggunakan Drizzle ORM + mysql2                                   | Final  |
| AD-011 | Authentication menggunakan short-lived access token dan refresh session    | Final  |
| AD-012 | Authorization menggunakan Role + Resource Scope                            | Final  |
| AD-013 | Password disimpan menggunakan secure password hashing                      | Final  |
| AD-014 | Internal entity identifier menggunakan UUIDv7                              | Final  |
| AD-015 | Petugas langsung mengisi form tanpa wajib mencari keluarga terlebih dahulu | Final  |
| AD-016 | Sistem melakukan duplicate/family check ringan setelah form disimpan       | Final  |
| AD-017 | Kandidat keluarga tidak pernah di-merge otomatis jika belum pasti          | Final  |
| AD-018 | Satu keluarga dapat memiliki lebih dari satu pembayaran                    | Final  |
| AD-019 | Admin Desa tidak memiliki direct payment override pada V1                  | Final  |
| AD-020 | Koreksi setelah approval menggunakan `CORRECTION_REQUIRED`                 | Final  |
| AD-021 | Rekap dihitung dari pembayaran sumber                                      | Final  |
| AD-022 | Audit trail digunakan untuk aktivitas penting                              | Final  |
| AD-023 | V1 menggunakan online-first                                                | Final  |
| AD-024 | Offline operational write dapat dikembangkan kemudian                      | Final  |
| AD-025 | Workflow resmi tetap online-only                                           | Final  |
| AD-026 | V1 tidak menggunakan microservices                                         | Final  |
| AD-027 | V1 tidak menggunakan Kubernetes                                            | Final  |
| AD-028 | Project menggunakan satu repository                                        | Final  |

---

# 4. High-Level Architecture

```text
┌─────────────────────────────────────────┐
│          FLUTTER MOBILE APP             │
│                                         │
│   Petugas Lokasi                        │
│   Admin Desa                            │
│   DKM                                   │
│                                         │
│   UI & fitur berdasarkan role           │
└────────────────────┬────────────────────┘
                     │
                     │ HTTPS
                     │ REST API
                     ▼
┌─────────────────────────────────────────┐
│               BACKEND API               │
│                                         │
│   Node.js + TypeScript + Express.js     │
│                                         │
│   Authentication                        │
│   Authorization                         │
│   Business Logic                        │
│   Workflow                              │
│   Aggregation                           │
│   Reporting                             │
│   Audit                                 │
└────────────────────┬────────────────────┘
                     │
                     │ Drizzle ORM
                     │ mysql2
                     ▼
               ┌──────────────┐
               │    MySQL     │
               │              │
               │ Persistent   │
               │ Data Store   │
               └──────────────┘
```

---

# 5. Client-Server Architecture

Sistem menggunakan arsitektur client-server.

Flutter berfungsi sebagai client.

Backend menjadi pusat:

* authentication;
* authorization;
* validation;
* business rules;
* workflow;
* persistence;
* aggregation;
* reporting;
* audit.

Flutter tidak berkomunikasi langsung dengan database.

```text
Flutter
   │
   ▼
REST API
   │
   ▼
Backend
   │
   ▼
MySQL
```

---

# 6. Mobile Application

Sistem V1 hanya memiliki satu aplikasi client yaitu Flutter Mobile Application.

Aplikasi digunakan oleh:

* Petugas Lokasi;
* Admin Desa;
* DKM.

Setelah login, aplikasi menampilkan interface sesuai role pengguna.

```text
Login
  │
  ▼
Current User
  │
  ├── PETUGAS
  │      ↓
  │   Dashboard Petugas
  │
  ├── ADMIN_DESA
  │      ↓
  │   Dashboard Admin
  │
  └── DKM
         ↓
      Dashboard DKM
```

Role di Flutter digunakan untuk menyesuaikan UI.

Authorization tetap ditentukan oleh backend.

---

# 7. Flutter Architecture

Flutter menggunakan pendekatan:

> Feature-First Architecture dengan separation of concerns.

Struktur konseptual:

```text
lib/
│
├── app/
│
├── core/
│
└── features/
    ├── auth/
    ├── dashboard/
    ├── users/
    ├── locations/
    ├── periods/
    ├── families/
    ├── payments/
    ├── recaps/
    ├── reports/
    └── audit/
```

---

# 8. Feature Structure

Setiap feature dapat menggunakan struktur:

```text
payments/
│
├── data/
│   ├── datasources/
│   ├── models/
│   └── repositories/
│
├── domain/
│   ├── entities/
│   ├── repositories/
│   └── usecases/
│
└── presentation/
    ├── pages/
    ├── widgets/
    └── state/
```

Struktur dapat disederhanakan apabila suatu feature sangat kecil.

Clean Architecture tidak digunakan secara dogmatis.

Tujuan utamanya adalah menjaga:

* UI;
* state;
* business representation;
* networking;
* persistence;

tidak tercampur.

---

# 9. Riverpod

Flutter menggunakan **Riverpod** sebagai:

* state management;
* dependency management;
* asynchronous state management.

Riverpod bertanggung jawab terhadap state aplikasi seperti:

* authentication state;
* current user;
* dashboard;
* daftar pembayaran;
* form state;
* loading;
* error;
* refresh;
* repository dependency.

Konsep:

```text
Flutter UI
    │
    │ ref.watch(...)
    ▼
Provider / Notifier
    │
    ▼
Use Case / Repository
    │
    ▼
Data Source
    │
    ▼
REST API
```

---

# 10. Riverpod Provider Strategy

Jenis Riverpod yang diutamakan:

### Provider

Untuk dependency relatif statis.

Contoh:

```text
dioProvider
paymentRepositoryProvider
authRepositoryProvider
```

### Notifier

Untuk synchronous mutable state.

### AsyncNotifier

Untuk asynchronous mutable state.

Contoh:

```text
AuthController
PaymentController
RecapController
```

### FutureProvider

Untuk asynchronous read-only sederhana jika memang sesuai.

---

# 11. Riverpod Principles

Riverpod tidak menjadi tempat business rule authoritative.

Contoh:

Flutter boleh menyembunyikan tombol Approve untuk Petugas.

Tetapi backend tetap wajib menolak:

```text
Petugas
   ↓
POST /recaps/{id}/approve
   ↓
403 Forbidden
```

Riverpod menangani application/client state.

Backend menangani:

* authorization;
* business rules;
* workflow validity.

---

# 12. Riverpod Code Generation

Riverpod code generation tidak diwajibkan pada tahap awal V1.

Project dapat menggunakan Riverpod tanpa generator terlebih dahulu.

Code generation dapat dipertimbangkan di masa depan apabila memberikan manfaat nyata.

---

# 13. Mobile Networking

Flutter menggunakan HTTP client untuk mengakses API.

Kandidat implementation:

```text
Dio
```

Networking layer menangani:

* base URL;
* timeout;
* authentication header;
* request;
* response;
* error mapping;
* refresh token mechanism.

UI tidak boleh memanggil `Dio` secara langsung.

---

# 14. Mobile Repository Pattern

Alur:

```text
UI
 ↓
Riverpod Controller
 ↓
Use Case
 ↓
Repository
 ↓
Remote Data Source
 ↓
REST API
```

Contoh:

```text
PaymentFormPage
      ↓
CreatePaymentController
      ↓
CreatePaymentUseCase
      ↓
PaymentRepository
      ↓
PaymentRemoteDataSource
      ↓
Backend API
```

---

# 15. Backend Architecture

Backend menggunakan:

> Modular Monolith.

Seluruh backend berjalan sebagai satu aplikasi tetapi dipisahkan menjadi domain module.

```text
backend/
└── src/
    ├── core/
    ├── config/
    ├── middleware/
    │
    └── modules/
        ├── auth/
        ├── users/
        ├── locations/
        ├── periods/
        ├── families/
        ├── payments/
        ├── recaps/
        ├── reports/
        └── audit/
```

---

# 16. Kenapa Modular Monolith?

Dipilih karena:

* skala sistem tingkat desa;
* tim kecil;
* deployment sederhana;
* transaction database lebih mudah;
* debugging lebih mudah;
* testing lebih sederhana;
* belum membutuhkan distributed system.

Microservices akan menambahkan kompleksitas yang tidak diperlukan pada V1.

---

# 17. Backend Layering

Backend menggunakan konsep:

```text
HTTP Request
     ↓
Route
     ↓
Controller
     ↓
Application / Use Case
     ↓
Domain / Business Rule
     ↓
Repository
     ↓
Database
```

Controller harus relatif tipis.

Business logic tidak ditempatkan langsung di route/controller.

---

# 18. Application Use Cases

Contoh application use case:

```text
Login
CreatePayment
UpdatePayment
CancelPayment
SubmitRecap
RequestRevision
ApproveRecap
RequestCorrection
ResubmitRecap
ClosePeriod
```

Application layer melakukan orchestration.

---

# 19. Domain Layer

Domain layer menyimpan aturan dan konsep bisnis seperti:

* Family;
* Payment;
* Period;
* Location;
* Recap;
* RecapStatus;
* UserRole.

Contoh business rule:

```text
ricePeople + moneyPeople = totalPeople
```

atau:

```text
SUBMITTED tidak dapat diedit
```

---

# 20. Backend Technology

Stack backend V1:

```text
Runtime       : Node.js
Language      : TypeScript
Framework     : Express.js
Architecture  : Modular Monolith
API           : REST
Database      : MySQL
ORM           : Drizzle ORM
Driver        : mysql2
```

---

# 21. Kenapa TypeScript?

TypeScript dipilih untuk:

* type safety;
* refactoring;
* contract antar layer;
* maintainability;
* mengurangi runtime error sederhana;
* mempermudah pengembangan jangka panjang.

---

# 22. Database

Database utama menggunakan MySQL.

Relational database dipilih karena sistem memiliki hubungan data yang kuat dan membutuhkan:

* foreign key;
* constraints;
* transaction;
* aggregation;
* indexing;
* historical data.

---

# 23. Drizzle ORM

Backend menggunakan:

> Drizzle ORM + mysql2.

Drizzle dipilih karena:

* type-safe;
* SQL-like;
* abstraksi relatif tipis;
* kontrol query tinggi;
* mendukung transactions;
* mendukung migrations;
* mudah mengoptimalkan query;
* cocok untuk aggregations dan reporting.

ORM tidak dianggap sebagai faktor utama scalability.

Scalability lebih banyak ditentukan oleh:

* database design;
* indexes;
* query design;
* pagination;
* connection management;
* infrastructure.

---

# 24. Database Migration

Schema database dikelola menggunakan migration.

Contoh:

```text
0001_create_users
0002_create_locations
0003_create_periods
0004_create_families
0005_create_payments
```

Perubahan schema production tidak dilakukan secara manual tanpa histori migration.

---

# 25. Identifier Strategy

Internal entity identifier menggunakan:

> UUIDv7

UUIDv7 digunakan untuk entity yang membutuhkan identifier global seperti:

* user;
* location;
* family;
* payment;
* recap;
* audit entry jika diperlukan.

Keuntungan:

* tidak bergantung pada data manusia;
* tidak konflik meskipun dibuat dari client berbeda;
* cocok untuk future offline write;
* tidak mengekspos urutan sederhana seperti auto increment kepada client.

---

# 26. Family Identification

UUIDv7 adalah technical identifier.

Petugas tidak menggunakan UUID untuk mengenali warga.

Informasi keluarga tetap dapat berisi:

* nama kepala keluarga;
* dusun;
* RT;
* RW;
* detail alamat apabila diperlukan.

Data tersebut digunakan untuk display dan duplicate checking, bukan sebagai primary identifier.

---

# 27. Family Input Strategy

Alur utama Petugas dibuat sesederhana mungkin.

Petugas **tidak diwajibkan mencari keluarga terlebih dahulu**.

Workflow:

```text
Warga datang
     ↓
Tambah Zakat
     ↓
Isi Form
     ↓
Klik Simpan
     ↓
Backend melakukan pengecekan ringan
```

Hal ini dipilih karena pembayaran lebih dari satu kali diperkirakan menjadi kasus yang relatif jarang.

Normal workflow tetap:

```text
Isi
↓
Simpan
↓
Selesai
```

---

# 28. Duplicate / Existing Family Detection

Setelah form valid, backend melakukan pengecekan ringan terhadap kemungkinan keluarga yang telah tercatat.

Pencarian dapat menggunakan kombinasi:

* normalized head name;
* dusun;
* RT;
* RW;
* periode;
* informasi lain yang relevan.

Pengecekan menggunakan database index agar tetap cepat.

---

# 29. Duplicate Check Performance

Duplicate checking tidak melakukan scanning seluruh database tanpa index.

Index akan dirancang pada Database Design.

Contoh logical index:

```text
normalized_head_name
rt
rw
period_id
```

Dengan index yang tepat, pencarian kandidat pada skala desa tidak diharapkan menjadi bottleneck.

Sistem tidak menggunakan:

* AI;
* machine learning;
* fuzzy matching berat;
* full database similarity scan;

pada V1.

---

# 30. Duplicate Handling

Jika tidak ditemukan kandidat:

```text
Create Family
+
Create Payment
```

Jika ditemukan kandidat:

```text
Candidate Found
      ↓
Flutter menampilkan konfirmasi
      ↓
┌───────────────┬────────────────┐
Same Family     Different Family
      │                 │
      ▼                 ▼
Add Payment       Create new Family
```

Sistem tidak boleh merge otomatis jika identitas tidak dapat dipastikan.

---

# 31. Multiple Payments

Satu keluarga dapat memiliki satu atau lebih pembayaran.

```text
Family
 │
 ├── Payment #1
 ├── Payment #2
 └── Payment #N
```

Pembayaran kedua atau berikutnya tidak menimpa pembayaran sebelumnya.

Walaupun kasus pembayaran berulang diperkirakan jarang, model ini dipertahankan agar sistem tidak rapuh terhadap pengecualian.

---

# 32. Mixed Payment

Satu pembayaran dapat menggunakan:

* beras;
* uang;
* kombinasi keduanya.

Contoh:

```text
Total Jiwa = 5

Beras:
3 jiwa
7.5 kg

Uang:
2 jiwa
Rp xxx.xxx
```

Validasi:

```text
rice_people + money_people = total_people
```

---

# 33. Authentication Architecture

Authentication menggunakan:

```text
Short-Lived Access Token
+
Refresh Session
```

Access token digunakan pada request API.

Refresh session digunakan untuk memperoleh access token baru.

---

# 34. Token Storage

Credential autentikasi disimpan di Flutter menggunakan secure storage.

Kandidat:

```text
flutter_secure_storage
```

Password tidak disimpan setelah login.

---

# 35. Password Security

Password tidak disimpan plaintext.

Backend menggunakan secure password hashing.

Rekomendasi:

```text
Argon2id
```

Konfigurasi final ditentukan pada Security Design.

---

# 36. Authorization

Authorization menggunakan dua dimensi:

```text
ROLE
+
RESOURCE SCOPE
```

Role:

```text
PETUGAS
ADMIN_DESA
DKM
```

---

# 37. Resource Scope

Petugas hanya boleh mengakses lokasi yang menjadi penugasannya.

Contoh:

```text
Petugas:
Location A

Request:
Payment Location B

Result:
403 Forbidden
```

Hal ini divalidasi backend.

---

# 38. Admin Override

V1 **tidak memberikan direct payment override kepada Admin Desa**.

Admin tidak dapat secara normal:

* membuat pembayaran atas nama Petugas;
* mengubah pembayaran lokasi;
* membatalkan pembayaran lokasi.

Jika ditemukan masalah:

```text
Admin
 ↓
Request Revision / Correction
 ↓
Petugas memperbaiki
 ↓
Submit ulang
 ↓
Admin memeriksa
```

Hal ini menjaga separation of responsibility.

---

# 39. Recap Architecture

Rekap Lokasi memiliki:

* location;
* period;
* workflow status;
* submission metadata;
* revision metadata;
* correction metadata;
* approval metadata.

Angka rekap berasal dari Family dan Payment sumber.

---

# 40. Recap Aggregation

Rekap minimal menghitung:

* jumlah KK;
* jumlah jiwa;
* jiwa beras;
* total beras;
* jiwa uang;
* total uang.

Satu Family dihitung satu KK walaupun memiliki banyak Payment.

---

# 41. Recap Source of Truth

Petugas tidak memasukkan total rekap secara manual.

```text
Families
+
Payments
    ↓
Aggregation Query
    ↓
Location Recap
```

Total harus dapat dihitung ulang dari source data.

---

# 42. Recap State Machine

State final V1:

```text
DRAFT
SUBMITTED
REVISION_REQUIRED
APPROVED
CORRECTION_REQUIRED
```

Normal workflow:

```text
DRAFT
  ↓ Submit
SUBMITTED
  │
  ├── Request Revision
  │        ↓
  │ REVISION_REQUIRED
  │        │
  │        └── Resubmit → SUBMITTED
  │
  └── Approve
           ↓
       APPROVED
```

---

# 43. Correction After Approval

Apabila kesalahan ditemukan setelah `APPROVED`:

```text
APPROVED
    ↓
Admin Request Correction
    ↓
CORRECTION_REQUIRED
    ↓
Petugas melakukan koreksi
    ↓
SUBMITTED
    ↓
Admin periksa ulang
    ↓
APPROVED
```

Admin wajib memasukkan alasan correction.

---

# 44. Revision vs Correction

`REVISION_REQUIRED`

digunakan ketika masalah ditemukan sebelum rekap pernah disetujui.

`CORRECTION_REQUIRED`

digunakan ketika masalah ditemukan setelah rekap sudah `APPROVED`.

---

# 45. Correction and Village Recap

Selama rekap berada pada:

```text
CORRECTION_REQUIRED
```

rekap tersebut tidak dianggap sebagai final approved recap untuk perhitungan resmi live.

Setelah diperbaiki dan disetujui kembali, rekap kembali masuk perhitungan resmi.

---

# 46. Recap Revision Number

Location Recap dapat memiliki:

```text
revision_no
```

Contoh:

```text
Revision 1
→ APPROVED

Correction

Revision 2
→ APPROVED
```

Detail penyimpanan revision akan ditentukan pada Database Design.

---

# 47. Editability by State

| Recap State         | Add Payment | Edit | Cancel |
| ------------------- | :---------: | :--: | :----: |
| DRAFT               |      ✅      |   ✅  |    ✅   |
| SUBMITTED           |      ❌      |   ❌  |    ❌   |
| REVISION_REQUIRED   |      ✅      |   ✅  |    ✅   |
| APPROVED            |      ❌      |   ❌  |    ❌   |
| CORRECTION_REQUIRED |      ✅      |   ✅  |    ✅   |

Backend wajib memeriksa status terbaru.

---

# 48. API Architecture

API menggunakan REST.

Base path:

```text
/api/v1
```

Resource utama secara konseptual:

```text
/api/v1/auth
/api/v1/users
/api/v1/locations
/api/v1/periods
/api/v1/families
/api/v1/payments
/api/v1/recaps
/api/v1/reports
/api/v1/audit-logs
```

Detail endpoint dibuat pada `14-api-design.md`.

---

# 49. Request Pipeline

Protected request secara umum:

```text
Request
   ↓
Request ID
   ↓
Rate Limit
   ↓
Authentication
   ↓
Authorization
   ↓
Input Validation
   ↓
Application Use Case
   ↓
Business Rules
   ↓
Repository
   ↓
Database
   ↓
Audit / Logging
   ↓
Response
```

---

# 50. Validation

Validation dilakukan di dua sisi.

### Flutter

Untuk memberikan feedback cepat kepada pengguna.

### Backend

Untuk menjaga:

* security;
* integrity;
* business rule.

Backend adalah authoritative validation layer.

---

# 51. Database Constraints

Database tetap menjadi lapisan integrity tambahan.

Contoh hubungan:

```text
Payment → Family

Family → Period

Payment → Location / operational context

Location Recap → Location + Period
```

Foreign key dan constraints detail ditentukan pada Database Design.

---

# 52. Transaction Management

Operasi kritis menggunakan database transaction.

Contoh:

```text
Create Family
+
Create Payment
+
Create Audit
```

Jika satu bagian gagal, operasi harus dapat di-roll back secara aman.

---

# 53. Duplicate Request Protection

Sistem harus melindungi create operation terhadap duplicate HTTP request.

Contoh:

```text
User tekan Simpan
↓
Internet lambat
↓
User tekan Simpan lagi
```

Payment menggunakan UUIDv7/client-generated identifier dan API dapat menggunakan idempotency key.

Server harus dapat mengenali request yang telah diproses.

---

# 54. Concurrency

Backend selalu memeriksa state terbaru.

Contoh:

```text
Flutter:
DRAFT

Server:
SUBMITTED
```

Jika pengguna mencoba Edit:

```text
Backend → Reject
```

Flutter state tidak menjadi authoritative source.

---

# 55. Optimistic Concurrency

Operasi state transition dapat menggunakan conditional update.

Contoh:

```text
UPDATE recap
SET status = 'APPROVED'
WHERE id = ?
AND status = 'SUBMITTED'
```

Jika tidak ada row yang berubah, operasi dianggap conflict.

---

# 56. Audit Architecture

Aktivitas penting disimpan pada audit log.

Minimal:

* actor;
* action;
* resource;
* timestamp.

Untuk perubahan penting dapat ditambahkan:

* before;
* after;
* reason.

Contoh:

```text
Admin A
REQUEST_CORRECTION
Recap X
Reason: ...
```

---

# 57. Application Logging

Application Log berbeda dari Audit Log.

Application Log:

```text
Database timeout
Unhandled exception
Slow API
```

Audit Log:

```text
Petugas A created Payment X.
Admin B approved Recap Y.
```

---

# 58. Reporting Architecture

Laporan dibuat oleh backend.

```text
Flutter
   ↓
Report Request
   ↓
Backend
   ├── Authorization
   ├── Query
   ├── Aggregate
   └── Generate
          ↓
       PDF / Excel
```

Flutter tidak menghitung laporan sendiri.

---

# 59. Report Security

Laporan harus mengikuti data minimization.

Laporan publik/pengumuman sebaiknya berisi data agregat seperti:

* jumlah KK;
* jumlah jiwa;
* total beras;
* total uang.

Identitas keluarga hanya dimasukkan apabila memang diperlukan untuk administrasi internal.

---

# 60. Security Architecture

Security menggunakan prinsip Defense in Depth.

```text
Flutter Secure Storage
        ↓
HTTPS
        ↓
Rate Limiting
        ↓
Authentication
        ↓
Authorization
        ↓
Validation
        ↓
Business Rules
        ↓
Database Constraints
        ↓
Audit
```

---

# 61. HTTPS

Semua komunikasi production wajib menggunakan HTTPS.

Data sensitif tidak boleh dikirim melalui plaintext HTTP.

---

# 62. Secrets

Secret backend seperti:

```text
DATABASE_PASSWORD
TOKEN_SECRET
```

tidak disimpan di Git repository.

Gunakan environment configuration.

Repository hanya menyimpan:

```text
.env.example
```

tanpa secret asli.

---

# 63. Database Access Security

Backend menggunakan database account khusus.

Production backend tidak menggunakan account `root`.

Database tidak dibuka ke public internet tanpa kebutuhan.

---

# 64. Rate Limiting

Endpoint sensitif seperti:

```text
/auth/login
/auth/refresh
```

menggunakan rate limiting.

---

# 65. Sensitive Logging

Application log tidak boleh mencatat:

* password;
* access token;
* refresh token;
* database credentials;
* application secrets.

---

# 66. Online-First Strategy

V1 menggunakan:

> Online-First Architecture.

Operasi write membutuhkan backend.

Contoh:

```text
Tambah Payment
Edit Payment
Cancel Payment
Submit Recap
Approve Recap
```

---

# 67. Future Offline Architecture

Arsitektur Flutter tetap dibuat agar operational write dapat dikembangkan menjadi offline-capable.

Konsep:

```text
Flutter UI
    ↓
Riverpod
    ↓
Repository
    │
    ├── Remote Data Source
    │       ↓
    │      API
    │
    └── Local Data Source
            ↓
          SQLite
```

UI tidak perlu mengetahui sumber data yang sedang digunakan.

---

# 68. Offline Operational Write

Di masa depan operasi berikut dapat dibuat offline-capable:

* tambah Payment;
* edit draft;
* cancel draft;
* melihat cached data.

Saat offline:

```text
Create Payment
     ↓
Local Database
     ↓
PENDING_SYNC
```

Ketika internet kembali:

```text
PENDING_SYNC
     ↓
Sync Engine
     ↓
Backend
     ↓
SYNCED
```

---

# 69. Workflow Operations Remain Online

Operasi berikut tetap online-only pada desain awal:

* Submit Rekap;
* Request Revision;
* Approve;
* Request Correction;
* Close Period.

Karena operasi tersebut memerlukan state server terbaru.

---

# 70. Offline Sync Safety

UUIDv7 membantu future offline sync.

Payment dapat dibuat di device dengan UUID unik sebelum dikirim ke server.

Idempotency mechanism memastikan retry tidak menghasilkan data ganda.

---

# 71. Local Database Candidate

Jika offline capability dikembangkan, kandidat local relational database adalah:

```text
SQLite
```

dengan Flutter persistence library seperti Drift atau library lain yang sesuai saat implementasi.

Belum diperlukan pada V1 online-first.

---

# 72. Deployment Architecture

Deployment awal:

```text
Internet
   ↓
HTTPS Endpoint
   ↓
Reverse Proxy / Platform Gateway
   ↓
Backend Application
   ↓
MySQL
```

Tidak menggunakan Kubernetes.

---

# 73. Docker

Backend dirancang dapat dijalankan melalui Docker.

Tujuan:

* environment konsisten;
* deployment repeatable;
* dependency terisolasi.

---

# 74. Repository Structure

Project menggunakan monorepository sederhana.

```text
zakat-desa/
│
├── mobile/
├── backend/
├── docs/
├── diagrams/
├── docker/
├── .github/
│   └── workflows/
├── README.md
└── .gitignore
```

---

# 75. Environment

Minimal terdapat:

```text
Development
Testing
Production
```

Konfigurasi environment tidak di-hard-code.

---

# 76. Testing Architecture

Arsitektur harus mendukung:

* Unit Test;
* Repository Test;
* API Integration Test;
* Flutter Widget Test;
* Integration Test;
* End-to-End Test.

Business rule kritis harus mudah diuji.

---

# 77. Performance Strategy

Optimasi utama V1:

* database indexes;
* pagination;
* efficient query;
* connection management;
* aggregate query yang tepat.

Tidak menggunakan caching kompleks sebelum terbukti diperlukan.

---

# 78. Pagination

Resource yang dapat berkembang menggunakan pagination.

Contoh:

* families;
* payments;
* users;
* audit logs.

Aplikasi tidak mengambil seluruh dataset tanpa kebutuhan.

---

# 79. Scalability Strategy

V1 dapat berjalan dengan:

```text
1 Backend Instance
+
1 MySQL Database
```

Ini memadai untuk kebutuhan awal tingkat desa.

Jika kebutuhan meningkat, backend dapat dikembangkan menjadi beberapa instance tanpa langsung mengubah domain menjadi microservices.

---

# 80. Extensibility

Modular architecture memungkinkan penambahan domain masa depan seperti:

```text
Zakat Mal
Mustahik
Distribution
```

tanpa harus membangun ulang aplikasi dari awal.

V1 tetap hanya menangani Zakat Fitrah.

---

# 81. Architecture Principles

## AP-001 — Simplicity First

Gunakan solusi paling sederhana yang memenuhi kebutuhan.

## AP-002 — Backend Is Authoritative

Backend menjadi source of truth terhadap permission, workflow, dan business rule.

## AP-003 — Preserve History

Pembayaran penting tidak dihapus destruktif.

## AP-004 — Least Privilege

Pengguna hanya memperoleh hak yang diperlukan.

## AP-005 — Derived Recap

Rekap berasal dari source data.

## AP-006 — Human Confirmation

Sistem membantu duplicate detection tetapi tidak melakukan merge keluarga secara otomatis jika tidak pasti.

## AP-007 — Simple Operational UX

Petugas tidak dibebani pencarian manual sebelum setiap input.

## AP-008 — Modular Design

Domain dipisahkan secara logis.

## AP-009 — Testability

Business logic dapat diuji secara independen dari Flutter atau HTTP layer.

## AP-010 — Secure by Design

Security menjadi bagian arsitektur sejak awal.

## AP-011 — Avoid Premature Complexity

Microservices, message broker, Kubernetes, AI matching, dan offline synchronization tidak digunakan sebelum dibutuhkan.

## AP-012 — Evolution Instead of Rewrite

Desain memungkinkan fitur baru ditambahkan tanpa membangun ulang seluruh sistem.

---

# 82. Final Architecture Overview

```text
┌──────────────────────────────────────────┐
│           FLUTTER MOBILE APP             │
│                                          │
│   Petugas | Admin Desa | DKM             │
│                                          │
│   Presentation                           │
│        ↓                                 │
│   Riverpod                               │
│        ↓                                 │
│   Use Case / Repository                  │
│        ↓                                 │
│   Remote Data Source                     │
│                                          │
│   Secure Authentication Storage          │
└─────────────────────┬────────────────────┘
                      │
                      │ HTTPS / REST
                      ▼
┌──────────────────────────────────────────┐
│              BACKEND API                 │
│                                          │
│ Node.js + TypeScript + Express.js        │
│                                          │
│ Middleware                               │
│ ├── Request ID                           │
│ ├── Rate Limiting                        │
│ ├── Authentication                       │
│ ├── Authorization                        │
│ └── Validation                           │
│                                          │
│ Modular Application                      │
│ ├── Auth                                 │
│ ├── Users                                │
│ ├── Locations                            │
│ ├── Periods                              │
│ ├── Families                             │
│ ├── Payments                             │
│ ├── Recaps                               │
│ ├── Reports                              │
│ └── Audit                                │
│                                          │
│ Application → Domain → Repository        │
└─────────────────────┬────────────────────┘
                      │
               Drizzle ORM + mysql2
                      │
                      ▼
               ┌──────────────┐
               │    MySQL     │
               │              │
               │ UUIDv7 IDs   │
               │ Foreign Keys │
               │ Constraints  │
               │ Indexes      │
               │ Transactions │
               └──────────────┘
```

---

# 83. Final Technology Stack

| Area                 | Technology                                       |
| -------------------- | ------------------------------------------------ |
| Mobile               | Flutter                                          |
| Language Mobile      | Dart                                             |
| State Management     | Riverpod                                         |
| HTTP Client          | Dio                                              |
| Secure Storage       | Android-backed secure storage                    |
| Backend Runtime      | Node.js                                          |
| Backend Language     | TypeScript                                       |
| Backend Framework    | Express.js                                       |
| Backend Architecture | Modular Monolith                                 |
| API                  | REST                                             |
| ORM                  | Drizzle ORM                                      |
| MySQL Driver         | mysql2                                           |
| Database             | MySQL                                            |
| Entity Identifier    | UUIDv7                                           |
| Authentication       | Access Token + Refresh Session                   |
| Password Hashing     | Argon2id recommended                             |
| Communication        | HTTPS                                            |
| Deployment           | Docker-capable                                   |
| Offline Strategy     | Online-first, future operational offline support |

---

# 84. Resolved Architecture Decisions

Keputusan berikut tidak lagi berstatus pending:

### State Management

Riverpod.

### ORM

Drizzle ORM + mysql2.

### Admin Override

Tidak ada direct payment override pada V1.

### Correction After Approval

Menggunakan:

`APPROVED → CORRECTION_REQUIRED → SUBMITTED → APPROVED`

### Family Identifier

Internal entity identifier menggunakan UUIDv7.

Nama/wilayah digunakan untuk pencarian dan duplicate detection, bukan sebagai primary identifier.

### Family Input

Petugas langsung mengisi form.

Sistem melakukan duplicate check ringan setelah form disimpan.

### Multiple Payment

Didukung sebagai safety net walaupun diperkirakan jarang terjadi.

### Offline

V1 online-first.

Offline operational write dapat ditambahkan kemudian tanpa mengubah keseluruhan architecture.

---

# 85. Remaining Decisions

Keputusan yang masih dapat ditentukan pada tahap berikutnya:

1. Struktur tabel dan foreign key final.
2. Index duplicate checking final.
3. Apakah Family bersifat global tingkat desa atau registrasi per periode.
4. Schema refresh session.
5. Schema recap revision.
6. Detail audit metadata.
7. ORM schema implementation.
8. Endpoint API detail.
9. Token expiration duration.
10. Deployment provider.

Keputusan tersebut akan dibahas pada Database Design, API Design, Security Design, dan Deployment Design.

---

# 86. Conclusion

Arsitektur final V1 menggunakan:

```text
Flutter + Riverpod
        ↓
      HTTPS
        ↓
REST API
Node.js + TypeScript + Express.js
Modular Monolith
        ↓
Drizzle ORM + mysql2
        ↓
       MySQL
```

Seluruh pengguna menggunakan satu aplikasi Flutter.

Backend menjadi authoritative source untuk:

* authentication;
* authorization;
* validation;
* business rules;
* workflow;
* aggregation;
* reporting;
* audit.

Family dan Payment dipisahkan sehingga satu keluarga dapat memiliki beberapa riwayat pembayaran tanpa menduplikasi jumlah KK.

UUIDv7 digunakan sebagai identifier internal.

Petugas tidak diwajibkan mencari keluarga sebelum input. Sistem melakukan duplicate checking ringan setelah form diisi dan hanya meminta konfirmasi apabila terdapat kandidat keluarga yang mungkin sama.

Workflow rekap menggunakan:

`DRAFT`

→ `SUBMITTED`

→ `REVISION_REQUIRED` atau `APPROVED`

dan apabila terdapat kesalahan setelah approval:

`APPROVED`

→ `CORRECTION_REQUIRED`

→ `SUBMITTED`

→ `APPROVED`.

Admin Desa tidak melakukan direct edit terhadap pembayaran Petugas.

V1 menggunakan online-first, tetapi repository architecture, UUIDv7, dan idempotency strategy memungkinkan offline operational write dikembangkan di masa depan tanpa melakukan rewrite besar terhadap sistem.

Dokumen ini menjadi dasar untuk tahap berikutnya yaitu `13-database-design.md`.
