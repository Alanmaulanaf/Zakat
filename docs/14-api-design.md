# API Design

## 1. Tujuan Dokumen

Dokumen ini mendefinisikan desain REST API untuk Sistem Pendataan dan Rekapitulasi Zakat Fitrah Desa V1.

API menjadi kontrak komunikasi antara:

```text
Flutter Mobile Application
        ↓
      HTTPS
        ↓
REST API Backend
        ↓
      MySQL
```

API Design mencakup:

* API versioning;
* endpoint structure;
* authentication;
* authorization;
* request dan response format;
* validation;
* family detection;
* payment processing;
* recap workflow;
* reporting;
* pagination;
* filtering;
* idempotency;
* concurrency;
* error handling;
* security;
* audit integration.

---

# 2. Status API Design

Dokumen ini menjadi **baseline API V1**.

Final tidak berarti API tidak dapat dikembangkan.

Perubahan API dapat dilakukan apabila:

* requirement berubah;
* database design berkembang;
* kebutuhan mobile berubah;
* offline synchronization dikembangkan;
* ditemukan kebutuhan security baru;
* fitur baru ditambahkan.

Perubahan breaking harus mempertimbangkan API versioning.

---

# 3. API Architecture

API menggunakan:

```text
Style          : REST
Transport      : HTTPS
Data Format    : JSON
API Version    : v1
Authentication : Bearer Access Token
```

Base URL secara konseptual:

```text
https://api.example.com/api/v1
```

Development dapat menggunakan URL yang berbeda.

---

# 4. API Versioning

Seluruh endpoint V1 menggunakan prefix:

```text
/api/v1
```

Contoh:

```text
/api/v1/auth/login
/api/v1/payments
/api/v1/location-recaps
```

Jika suatu saat terdapat breaking change besar:

```text
/api/v2
```

dapat diperkenalkan tanpa langsung merusak aplikasi V1.

---

# 5. Naming Convention

URL menggunakan:

```text
kebab-case
```

Contoh:

```text
/zakat-periods
/location-recaps
/audit-logs
```

JSON menggunakan:

```text
camelCase
```

Contoh:

```json
{
  "familyId": "...",
  "paymentDate": "2027-03-20",
  "totalPeople": 5,
  "ricePeople": 3,
  "riceKg": "7.500",
  "moneyPeople": 2,
  "moneyAmount": 100000
}
```

Database tetap menggunakan:

```text
snake_case
```

Contoh mapping:

```text
Database:
payment_date

API:
paymentDate
```

---

# 6. UUID Representation

Database menyimpan UUIDv7 menggunakan:

```text
BINARY(16)
```

Tetapi API mengirim UUID sebagai string.

Contoh:

```json
{
  "id": "019d1f1a-93cf-7c01-8b90-..."
}
```

Flutter tidak perlu mengetahui representasi `BINARY(16)` di database.

---

# 7. Date and Time Format

Business date menggunakan:

```text
YYYY-MM-DD
```

Contoh:

```json
{
  "paymentDate": "2027-03-20"
}
```

Timestamp menggunakan ISO 8601 UTC.

Contoh:

```json
{
  "createdAt": "2027-03-20T09:30:15.123Z"
}
```

Flutter mengubah timestamp menjadi local timezone untuk display.

---

# 8. Numeric Representation

Nominal Rupiah dikirim sebagai integer.

Contoh:

```json
{
  "moneyAmount": 100000
}
```

Nilai desimal seperti berat beras direkomendasikan dikirim sebagai decimal string.

Contoh:

```json
{
  "riceKg": "7.500"
}
```

Tujuannya menghindari masalah floating-point precision antar platform.

---

# 9. Standard Success Response

Response JSON normal menggunakan struktur:

```json
{
  "success": true,
  "data": {},
  "meta": {}
}
```

`meta` bersifat opsional.

Contoh:

```json
{
  "success": true,
  "data": {
    "id": "019d..."
  }
}
```

---

# 10. Standard Error Response

Error menggunakan struktur:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Data yang dikirim tidak valid.",
    "details": [],
    "requestId": "req_..."
  }
}
```

`code` menggunakan Bahasa Inggris agar mudah diproses aplikasi.

`message` dapat menggunakan Bahasa Indonesia sebagai pesan yang mudah dipahami pengguna.

---

# 11. HTTP Status Code

| Status | Penggunaan                                         |
| ------ | -------------------------------------------------- |
| `200`  | Request berhasil                                   |
| `201`  | Resource berhasil dibuat                           |
| `204`  | Berhasil tanpa response body                       |
| `400`  | Request tidak dapat diproses secara sintaksis      |
| `401`  | Belum terautentikasi                               |
| `403`  | Tidak memiliki authorization                       |
| `404`  | Resource tidak ditemukan                           |
| `409`  | Conflict / workflow conflict / family confirmation |
| `422`  | Validation atau business rule gagal                |
| `429`  | Rate limit                                         |
| `500`  | Internal server error                              |

---

# 12. Standard Error Codes

Initial machine-readable error codes:

```text
VALIDATION_ERROR
AUTHENTICATION_REQUIRED
INVALID_CREDENTIALS
SESSION_EXPIRED
SESSION_REVOKED
FORBIDDEN
RESOURCE_NOT_FOUND
RESOURCE_CONFLICT
WORKFLOW_CONFLICT
FAMILY_CONFIRMATION_REQUIRED
DUPLICATE_REQUEST
INVALID_RECAP_STATE
PERIOD_NOT_ACTIVE
LOCATION_NOT_ACTIVE
ACCOUNT_INACTIVE
PAYMENT_ALREADY_CANCELLED
RATE_LIMIT_EXCEEDED
INTERNAL_ERROR
```

Error code dapat dikembangkan tanpa mengubah HTTP status convention.

---

# 13. Request ID

Setiap request memiliki:

```text
requestId
```

Backend dapat menghasilkan request ID jika client tidak mengirimkannya.

Optional header:

```text
X-Request-ID
```

Request ID digunakan untuk:

* error tracing;
* application logging;
* audit correlation;
* production debugging.

---

# 14. Authentication Header

Protected endpoint menggunakan:

```text
Authorization: Bearer <access_token>
```

Flutter mengambil token dari secure storage.

Backend tidak mempercayai role yang dikirim client.

Role diambil dari authenticated identity/server state.

---

# 15. Authentication Endpoints

## Login

```text
POST /api/v1/auth/login
```

Request:

```json
{
  "username": "petugas01",
  "password": "********"
}
```

Response:

```json
{
  "success": true,
  "data": {
    "accessToken": "...",
    "refreshToken": "...",
    "accessTokenExpiresAt": "2027-03-20T10:00:00Z",
    "user": {
      "id": "019d...",
      "name": "Petugas A",
      "username": "petugas01",
      "role": "PETUGAS"
    }
  }
}
```

---

# 16. Login Validation

Backend harus memeriksa:

```text
username exists
        ↓
password valid
        ↓
user is active
        ↓
authentication success
```

Error tidak boleh membocorkan apakah username atau password yang salah secara terlalu detail.

Response:

```text
INVALID_CREDENTIALS
```

cukup untuk login gagal.

---

# 17. Refresh Access Token

```text
POST /api/v1/auth/refresh
```

Request:

```json
{
  "refreshToken": "..."
}
```

Backend melakukan:

```text
Validate token
      ↓
Check refresh session
      ↓
Check revoked
      ↓
Check expiration
      ↓
Rotate / refresh session
      ↓
Return new access token
```

Detail token rotation ditentukan lebih lanjut pada Security Design.

---

# 18. Logout

```text
POST /api/v1/auth/logout
```

Refresh session yang digunakan harus direvoke.

Response:

```text
204 No Content
```

---

# 19. Current User

```text
GET /api/v1/auth/me
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "019d...",
    "name": "Petugas A",
    "username": "petugas01",
    "role": "PETUGAS",
    "isActive": true,
    "locations": [
      {
        "id": "019d...",
        "name": "Musholla Al-Ikhlas",
        "type": "MUSHOLLA"
      }
    ]
  }
}
```

Endpoint ini menjadi source utama Flutter untuk menentukan UI setelah login.

---

# 20. Authorization Model

Authorization menggunakan:

```text
ROLE
+
RESOURCE SCOPE
+
BUSINESS STATE
```

Contoh:

```text
Petugas
+
assigned to Location A
+
Recap DRAFT
```

maka dapat mengubah Payment Location A.

Tetapi:

```text
Petugas
+
Location B
```

menghasilkan:

```text
403 Forbidden
```

---

# 21. Role Overview

## `PETUGAS`

Dapat:

* melihat data lokasi yang ditugaskan;
* mencatat Payment;
* mengubah Payment ketika workflow memungkinkan;
* cancel Payment ketika workflow memungkinkan;
* melihat recap lokasi;
* submit recap;
* memperbaiki data revision/correction.

## `ADMIN_DESA`

Dapat:

* mengelola users;
* mengelola locations;
* mengelola periode;
* mengelola ketentuan zakat;
* memeriksa recap;
* request revision;
* approve;
* request correction;
* melihat village recap;
* melihat audit.

Admin tidak memiliki direct Payment override pada V1.

## `DKM`

Read-only terhadap:

* dashboard;
* recap;
* reports.

---

# 22. User Management API

Hanya `ADMIN_DESA`.

## List Users

```text
GET /api/v1/users
```

Optional query:

```text
?role=PETUGAS
&isActive=true
&search=ahmad
&limit=20
&cursor=...
```

---

# 23. Create User

```text
POST /api/v1/users
```

Request:

```json
{
  "name": "Budi",
  "username": "budi.petugas",
  "email": null,
  "password": "temporary-password",
  "role": "PETUGAS"
}
```

Password plaintext hanya digunakan selama request dan tidak boleh masuk log.

---

# 24. User Detail

```text
GET /api/v1/users/{userId}
```

---

# 25. Update User

```text
PATCH /api/v1/users/{userId}
```

Contoh:

```json
{
  "name": "Budi Santoso",
  "email": "budi@example.com"
}
```

Field yang dapat diubah harus menggunakan allowlist.

Client tidak boleh mengirim arbitrary database column.

---

# 26. Activate / Deactivate User

Menggunakan:

```text
PATCH /api/v1/users/{userId}
```

Contoh:

```json
{
  "isActive": false
}
```

Deactivation tidak menghapus histori.

---

# 27. Location Management API

Management dilakukan oleh `ADMIN_DESA`.

Read access dapat diberikan sesuai role.

## List Locations

```text
GET /api/v1/locations
```

Query:

```text
?type=MUSHOLLA
&isActive=true
&search=al-ikhlas
&limit=20
&cursor=...
```

---

# 28. Create Location

```text
POST /api/v1/locations
```

Request:

```json
{
  "name": "Musholla Al-Ikhlas",
  "type": "MUSHOLLA",
  "address": "Dusun ..."
}
```

---

# 29. Update Location

```text
PATCH /api/v1/locations/{locationId}
```

---

# 30. Location Assignment

Menugaskan Petugas:

```text
POST /api/v1/users/{userId}/location-assignments
```

Request:

```json
{
  "locationId": "019d..."
}
```

---

# 31. Update Assignment

```text
PATCH /api/v1/location-assignments/{assignmentId}
```

Contoh:

```json
{
  "isActive": false
}
```

Assignment historis tidak perlu di-hard-delete.

---

# 32. Zakat Period API

## List Periods

```text
GET /api/v1/zakat-periods
```

## Get Active Period

```text
GET /api/v1/zakat-periods/active
```

Endpoint ini penting untuk Flutter Petugas.

---

# 33. Create Period

`ADMIN_DESA`:

```text
POST /api/v1/zakat-periods
```

Request:

```json
{
  "year": 2027,
  "startDate": "2027-03-01",
  "endDate": "2027-03-31"
}
```

Initial status:

```text
DRAFT
```

---

# 34. Activate Period

```text
POST /api/v1/zakat-periods/{periodId}/activate
```

Backend memastikan tidak ada period `ACTIVE` lain.

---

# 35. Close Period

```text
POST /api/v1/zakat-periods/{periodId}/close
```

Backend harus memeriksa closing rules sebelum melakukan perubahan.

Jika masih ada recap yang tidak selesai dan rule melarang penutupan:

```text
409 WORKFLOW_CONFLICT
```

---

# 36. Zakat Period Rule

Get:

```text
GET /api/v1/zakat-periods/{periodId}/rules
```

Update:

```text
PUT /api/v1/zakat-periods/{periodId}/rules
```

Request:

```json
{
  "riceKgPerPerson": "2.500",
  "moneyAmountPerPerson": 50000,
  "notes": null
}
```

---

# 37. Family API

Family bukan resource yang normalnya dibuat langsung oleh Petugas.

Family creation terutama terjadi melalui:

```text
POST /payments
```

agar:

```text
Family
+
Registration
+
Payment
```

dapat diproses secara atomic.

---

# 38. List Families

```text
GET /api/v1/families
```

Digunakan ketika diperlukan untuk:

* melihat histori;
* pencarian;
* review;
* administrative inspection.

Query:

```text
?search=ahmad
&hamlet=manis
&rt=02
&rw=01
&limit=20
&cursor=...
```

Authorization tetap berlaku.

---

# 39. Family Detail

```text
GET /api/v1/families/{familyId}
```

Response dapat berisi:

```json
{
  "success": true,
  "data": {
    "id": "019d...",
    "headName": "Ahmad",
    "hamlet": "Manis",
    "rt": "02",
    "rw": "01",
    "addressDetail": null
  }
}
```

---

# 40. Update Family

Family data dapat diperbaiki melalui:

```text
PATCH /api/v1/families/{familyId}
```

Petugas hanya boleh melakukan perubahan jika:

* Family relevan dengan scope Petugas;
* current period workflow masih editable.

Admin tidak menggunakan endpoint ini sebagai direct Payment override.

Perubahan Family harus masuk audit.

---

# 41. Payment API

Payment merupakan transaksi operasional utama.

Endpoint create:

```text
POST /api/v1/payments
```

Endpoint ini juga menangani automatic family candidate detection.

---

# 42. Create Payment — Initial Request

Client membuat `paymentId` menggunakan UUIDv7 sebelum request dikirim.

Request:

```json
{
  "paymentId": "019d...",
  "family": {
    "headName": "Ahmad",
    "hamlet": "Manis",
    "rt": "02",
    "rw": "01",
    "addressDetail": null
  },
  "payment": {
    "paymentDate": "2027-03-20",
    "totalPeople": 5,
    "ricePeople": 3,
    "riceKg": "7.500",
    "moneyPeople": 2,
    "moneyAmount": 100000
  }
}
```

`locationId` normalnya tidak perlu diambil bebas dari input Petugas jika backend sudah mengetahui assigned location.

Hal ini mengurangi risiko Petugas memasukkan transaksi ke lokasi lain.

---

# 43. Create Payment — Backend Processing

Backend melakukan:

```text
Authentication
      ↓
Petugas authorization
      ↓
Assigned Location
      ↓
Active Period
      ↓
Recap editable?
      ↓
Validate payment
      ↓
Normalize family
      ↓
Find family candidate
```

---

# 44. Jika Tidak Ada Family Candidate

Backend melakukan transaction:

```text
Create Family
      ↓
Create Zakat Registration
      ↓
Create Payment
      ↓
Create Audit
```

Response:

```text
201 Created
```

Contoh:

```json
{
  "success": true,
  "data": {
    "payment": {
      "id": "019d...",
      "status": "ACTIVE"
    },
    "family": {
      "id": "019d...",
      "headName": "Ahmad"
    },
    "registration": {
      "id": "019d..."
    },
    "familyCreated": true
  }
}
```

---

# 45. Jika Family Candidate Ditemukan

Backend **tidak langsung membuat Payment**.

Response:

```text
409 Conflict
```

Error code:

```text
FAMILY_CONFIRMATION_REQUIRED
```

Contoh:

```json
{
  "success": false,
  "error": {
    "code": "FAMILY_CONFIRMATION_REQUIRED",
    "message": "Ditemukan data keluarga yang mungkin sama.",
    "requestId": "req_...",
    "details": {
      "confirmationToken": "temporary-token",
      "candidates": [
        {
          "familyId": "019d...",
          "headName": "Ahmad",
          "hamlet": "Manis",
          "rt": "02",
          "rw": "01",
          "addressDetail": null,
          "registeredInCurrentPeriod": true
        }
      ]
    }
  }
}
```

---

# 46. Candidate Data Minimization

Candidate response hanya menampilkan data yang diperlukan Petugas untuk konfirmasi.

Tidak boleh otomatis mengirim:

* seluruh histori pembayaran;
* audit data;
* authentication data;
* data sensitif lain.

Jika informasi pembayaran diperlukan sebagai pembeda, hanya summary minimum yang relevan yang boleh ditampilkan.

---

# 47. Confirm Existing Family

Jika Petugas memilih Family yang sama, request dikirim ulang:

```text
POST /api/v1/payments
```

dengan tambahan:

```json
{
  "paymentId": "019d...",
  "family": {
    "headName": "Ahmad",
    "hamlet": "Manis",
    "rt": "02",
    "rw": "01"
  },
  "payment": {
    "paymentDate": "2027-03-20",
    "totalPeople": 1,
    "ricePeople": 1,
    "riceKg": "2.500",
    "moneyPeople": 0,
    "moneyAmount": 0
  },
  "familyResolution": {
    "mode": "EXISTING",
    "familyId": "019d...",
    "confirmationToken": "temporary-token"
  }
}
```

Backend:

```text
Verify confirmation
      ↓
Verify Family
      ↓
Find/Create Registration
      ↓
Create new Payment
```

Payment lama tidak ditimpa.

---

# 48. Confirm New Family

Jika kandidat ternyata Family berbeda:

```json
{
  "familyResolution": {
    "mode": "NEW",
    "confirmationToken": "temporary-token"
  }
}
```

Backend membuat:

```text
New Family UUIDv7
+
New Registration
+
Payment
```

Keputusan tersebut dapat dicatat pada audit metadata.

---

# 49. Family Confirmation Token

`confirmationToken` bersifat:

* sementara;
* hanya untuk flow konfirmasi;
* tidak menggantikan access token;
* tidak menjadi authentication credential.

Detail implementasi token ditentukan pada Security Design.

---

# 50. Payment Validation

Backend memvalidasi:

```text
totalPeople >= 1
```

dan:

```text
ricePeople + moneyPeople = totalPeople
```

serta:

```text
ricePeople >= 0
moneyPeople >= 0
riceKg >= 0
moneyAmount >= 0
```

Minimal satu metode pembayaran harus bernilai valid.

---

# 51. List Payments

```text
GET /api/v1/payments
```

Petugas hanya memperoleh Payment sesuai assigned location.

Admin dapat memperoleh data sesuai administrative scope.

DKM tidak perlu mendapatkan detail Payment individual kecuali requirement read-only tertentu memang membutuhkannya.

Query:

```text
?periodId=...
&locationId=...
&status=ACTIVE
&paymentDate=2027-03-20
&search=ahmad
&limit=20
&cursor=...
```

Backend mengabaikan atau menolak scope query yang tidak diizinkan.

---

# 52. Payment Detail

```text
GET /api/v1/payments/{paymentId}
```

Response dapat mencakup:

* Payment;
* Family summary;
* Period;
* Location;
* recordedBy.

Authorization diperiksa berdasarkan resource, bukan hanya role.

---

# 53. Update Payment

```text
PATCH /api/v1/payments/{paymentId}
```

Contoh:

```json
{
  "paymentDate": "2027-03-20",
  "totalPeople": 4,
  "ricePeople": 4,
  "riceKg": "10.000",
  "moneyPeople": 0,
  "moneyAmount": 0,
  "expectedUpdatedAt": "2027-03-20T09:30:00.000Z"
}
```

`expectedUpdatedAt` digunakan untuk membantu optimistic concurrency.

---

# 54. Payment Update Authorization

Backend memeriksa:

```text
Authenticated?
      ↓
Petugas assigned to Payment Location?
      ↓
Period active?
      ↓
Current Location Recap editable?
      ↓
Payment ACTIVE?
      ↓
Update
```

Editable recap status:

```text
DRAFT
REVISION_REQUIRED
CORRECTION_REQUIRED
```

Tidak editable:

```text
SUBMITTED
APPROVED
```

---

# 55. Cancel Payment

Tidak menggunakan:

```text
DELETE /payments/{paymentId}
```

Normal cancellation menggunakan:

```text
POST /api/v1/payments/{paymentId}/cancel
```

Request:

```json
{
  "reason": "Data pembayaran tercatat dua kali."
}
```

Backend mengubah:

```text
ACTIVE
→
CANCELLED
```

dan menyimpan audit.

---

# 56. Payment Cancellation Conflict

Jika Payment sudah:

```text
CANCELLED
```

request berikutnya dapat menghasilkan:

```text
409 PAYMENT_ALREADY_CANCELLED
```

---

# 57. Location Recap API

List:

```text
GET /api/v1/location-recaps
```

Query:

```text
?periodId=...
&locationId=...
&status=SUBMITTED
&limit=20
&cursor=...
```

---

# 58. Location Recap Detail

```text
GET /api/v1/location-recaps/{recapId}
```

Response:

```json
{
  "success": true,
  "data": {
    "id": "019d...",
    "location": {
      "id": "019d...",
      "name": "Musholla Al-Ikhlas"
    },
    "period": {
      "id": "019d...",
      "year": 2027
    },
    "status": "DRAFT",
    "revisionNo": 0,
    "totals": {
      "totalFamilies": 100,
      "totalPeople": 350,
      "ricePeople": 250,
      "riceKg": "625.000",
      "moneyPeople": 100,
      "moneyAmount": 5000000
    }
  }
}
```

`totals` dihitung dari source Payment aktif.

---

# 59. Submit Recap

Petugas menggunakan:

```text
POST /api/v1/location-recaps/{recapId}/submit
```

Endpoint ini digunakan untuk:

```text
DRAFT → SUBMITTED
```

dan re-submit:

```text
REVISION_REQUIRED → SUBMITTED
```

serta:

```text
CORRECTION_REQUIRED → SUBMITTED
```

---

# 60. Submit Recap Processing

Backend melakukan transaction:

```text
Validate actor
      ↓
Validate current status
      ↓
Validate active period
      ↓
Calculate totals
      ↓
Validate recap not empty
      ↓
Increment revisionNo
      ↓
Create recap version snapshot
      ↓
Update status = SUBMITTED
      ↓
Workflow event
      ↓
Audit
```

---

# 61. Submit Recap Response

```json
{
  "success": true,
  "data": {
    "recapId": "019d...",
    "status": "SUBMITTED",
    "revisionNo": 2,
    "submittedAt": "2027-03-25T10:00:00Z"
  }
}
```

---

# 62. Request Revision

Hanya `ADMIN_DESA`.

```text
POST /api/v1/location-recaps/{recapId}/request-revision
```

Request:

```json
{
  "reason": "Jumlah jiwa pada salah satu keluarga perlu diperiksa."
}
```

Valid transition:

```text
SUBMITTED
→
REVISION_REQUIRED
```

`reason` wajib.

---

# 63. Approve Recap

Hanya `ADMIN_DESA`.

```text
POST /api/v1/location-recaps/{recapId}/approve
```

Valid transition:

```text
SUBMITTED
→
APPROVED
```

Backend harus memvalidasi current revision.

---

# 64. Request Correction

Hanya `ADMIN_DESA`.

```text
POST /api/v1/location-recaps/{recapId}/request-correction
```

Request:

```json
{
  "reason": "Ditemukan kesalahan setelah rekap disetujui."
}
```

Valid transition:

```text
APPROVED
→
CORRECTION_REQUIRED
```

---

# 65. Recap State Transition

| Current               | Action             | Role       | Next                  |
| --------------------- | ------------------ | ---------- | --------------------- |
| `DRAFT`               | Submit             | PETUGAS    | `SUBMITTED`           |
| `SUBMITTED`           | Request Revision   | ADMIN_DESA | `REVISION_REQUIRED`   |
| `SUBMITTED`           | Approve            | ADMIN_DESA | `APPROVED`            |
| `REVISION_REQUIRED`   | Submit             | PETUGAS    | `SUBMITTED`           |
| `APPROVED`            | Request Correction | ADMIN_DESA | `CORRECTION_REQUIRED` |
| `CORRECTION_REQUIRED` | Submit             | PETUGAS    | `SUBMITTED`           |

Transition lain ditolak.

---

# 66. Invalid Workflow Transition

Contoh:

```text
DRAFT
→ APPROVE
```

menghasilkan:

```text
409 INVALID_RECAP_STATE
```

Contoh response:

```json
{
  "success": false,
  "error": {
    "code": "INVALID_RECAP_STATE",
    "message": "Rekap belum dikirim dan tidak dapat disetujui.",
    "requestId": "req_..."
  }
}
```

---

# 67. Recap Versions

List versions:

```text
GET /api/v1/location-recaps/{recapId}/versions
```

Detail:

```text
GET /api/v1/location-recaps/{recapId}/versions/{revisionNo}
```

Digunakan untuk melihat historical submitted snapshot.

---

# 68. Recap Workflow History

```text
GET /api/v1/location-recaps/{recapId}/workflow-events
```

Response menampilkan:

```text
SUBMIT
REQUEST_REVISION
RESUBMIT
APPROVE
REQUEST_CORRECTION
```

sesuai histori.

---

# 69. Dashboard API

Satu endpoint:

```text
GET /api/v1/dashboard
```

Optional:

```text
?periodId=...
```

Backend mengembalikan response sesuai role authenticated user.

---

# 70. Petugas Dashboard Response

Contoh:

```json
{
  "success": true,
  "data": {
    "role": "PETUGAS",
    "period": {
      "id": "019d...",
      "year": 2027
    },
    "location": {
      "id": "019d...",
      "name": "Musholla Al-Ikhlas"
    },
    "recapStatus": "DRAFT",
    "totals": {
      "totalFamilies": 80,
      "totalPeople": 250,
      "riceKg": "500.000",
      "moneyAmount": 2500000
    }
  }
}
```

---

# 71. Admin Dashboard

Response dapat mencakup:

```text
total locations
DRAFT count
SUBMITTED count
REVISION_REQUIRED count
APPROVED count
CORRECTION_REQUIRED count
approved total families
approved total people
approved rice
approved money
```

---

# 72. DKM Dashboard

DKM memperoleh read-only aggregate data.

Tidak diberikan:

* create action;
* edit action;
* approval action;
* payment mutation endpoint.

---

# 73. Village Recap API

```text
GET /api/v1/reports/village-recap
```

Query:

```text
?periodId=...
```

Hanya current:

```text
APPROVED
```

Location Recaps yang masuk official aggregate.

---

# 74. Location Report API

```text
GET /api/v1/reports/location-recap
```

Query:

```text
?periodId=...
&locationId=...
```

Authorization mengikuti role dan scope.

---

# 75. Report Export

Location:

```text
GET /api/v1/reports/location-recap/export
```

Query:

```text
?periodId=...
&locationId=...
&format=pdf
```

Village:

```text
GET /api/v1/reports/village-recap/export
```

Query:

```text
?periodId=...
&format=xlsx
```

Format V1:

```text
pdf
xlsx
```

---

# 76. Export Response

Response file menggunakan appropriate:

```text
Content-Type
Content-Disposition
```

Contoh PDF:

```text
Content-Type: application/pdf
```

Excel:

```text
application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
```

Report generation tidak mengubah source data.

---

# 77. Audit API

Hanya role yang memiliki administrative authorization.

```text
GET /api/v1/audit-logs
```

Query:

```text
?actorId=...
&resourceType=PAYMENT
&resourceId=...
&action=UPDATE_PAYMENT
&from=...
&to=...
&limit=20
&cursor=...
```

---

# 78. Audit Detail

```text
GET /api/v1/audit-logs/{auditId}
```

Sensitive data tetap harus difilter sebelum response.

---

# 79. Pagination Strategy

List endpoint menggunakan cursor-based pagination.

Request:

```text
?limit=20
&cursor=opaque_cursor_value
```

Response:

```json
{
  "success": true,
  "data": [
    {}
  ],
  "meta": {
    "limit": 20,
    "nextCursor": "opaque_cursor",
    "hasMore": true
  }
}
```

---

# 80. Kenapa Cursor Pagination?

Cursor dipilih karena:

* lebih stabil ketika data baru masuk;
* cocok untuk dataset yang terus bertambah;
* lebih baik untuk future offline/sync style;
* tidak bergantung pada offset yang semakin besar.

Cursor harus dianggap opaque oleh Flutter.

Flutter tidak boleh mencoba membaca isi cursor.

---

# 81. Default Pagination Limit

Contoh baseline:

```text
default = 20
maximum = 100
```

Nilai final dapat disesuaikan berdasarkan testing.

Client tidak boleh meminta:

```text
limit=100000
```

untuk mengambil seluruh database.

---

# 82. Search Strategy

Search endpoint harus menggunakan query terkontrol.

Contoh:

```text
GET /families?search=ahmad
```

Backend menentukan field mana yang dapat dicari.

Client tidak dapat mengirim SQL-like condition.

Tidak boleh ada API seperti:

```text
?where=...
```

yang langsung diterjemahkan menjadi arbitrary SQL.

---

# 83. Filtering

Filter harus menggunakan explicit allowlist.

Contoh Payment:

```text
periodId
locationId
status
paymentDate
search
```

Contoh Recap:

```text
periodId
locationId
status
```

Unknown atau unsupported filter dapat diabaikan atau ditolak secara konsisten.

---

# 84. Sorting

Sorting dibatasi pada field yang disetujui.

Contoh:

```text
?sort=createdAt
&order=desc
```

Client tidak boleh menentukan arbitrary SQL field.

Default ordering list transaction:

```text
createdAt DESC
```

dengan secondary stable ordering:

```text
id DESC
```

---

# 85. Idempotency

Create Payment harus aman terhadap duplicate request.

Client membuat:

```text
paymentId = UUIDv7
```

sebelum mengirim request.

Optional header tambahan:

```text
Idempotency-Key: <uuid>
```

---

# 86. Duplicate Network Retry

Contoh:

```text
Flutter sends Payment A
        ↓
Server saves successfully
        ↓
Network drops before response
        ↓
Flutter retries Payment A
```

Server melihat:

```text
paymentId already exists
```

dan tidak membuat Payment kedua.

---

# 87. Idempotency Response

Retry terhadap operasi yang sama sebaiknya menghasilkan hasil yang konsisten.

Jika `paymentId` sama tetapi payload berbeda secara material:

```text
409 RESOURCE_CONFLICT
```

Ini mencegah identifier yang sama digunakan untuk dua transaksi berbeda.

---

# 88. Optimistic Concurrency

Update mutable resource dapat membawa:

```text
expectedUpdatedAt
```

Backend membandingkan dengan value terbaru.

Jika data sudah berubah:

```text
409 RESOURCE_CONFLICT
```

Flutter kemudian harus melakukan refresh data.

---

# 89. Workflow Concurrency

State transition selalu menggunakan current database state.

Contoh:

```text
Admin A membuka recap SUBMITTED
Admin B approve terlebih dahulu
Admin A kemudian request revision
```

Backend melihat current status:

```text
APPROVED
```

sehingga request revision ditolak.

UI state tidak menjadi source of truth.

---

# 90. Server-Side Validation

Flutter validation digunakan untuk UX.

Backend tetap wajib melakukan validation ulang.

Contoh payload berbahaya:

```json
{
  "totalPeople": -10
}
```

harus ditolak meskipun Flutter normalnya tidak pernah mengirim nilai tersebut.

---

# 91. Validation Error Detail

Contoh:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Data pembayaran tidak valid.",
    "details": [
      {
        "field": "totalPeople",
        "message": "Jumlah jiwa minimal 1."
      }
    ],
    "requestId": "req_..."
  }
}
```

Flutter dapat menggunakan `field` untuk menampilkan pesan pada form yang sesuai.

---

# 92. Security Headers and Transport

Production API hanya tersedia melalui:

```text
HTTPS
```

Request authentication tidak diperbolehkan melalui plaintext HTTP.

Reverse proxy/platform deployment dapat menangani TLS termination.

---

# 93. Rate Limiting

Endpoint sensitif memiliki rate limit.

Prioritas:

```text
POST /auth/login
POST /auth/refresh
```

Endpoint mutation lainnya juga dapat memiliki generic rate limit.

Response:

```text
429 Too Many Requests
```

---

# 94. Sensitive Data Handling

API tidak pernah mengembalikan:

```text
passwordHash
tokenHash
database credentials
application secrets
```

Refresh token hanya diberikan dalam flow authentication yang memang membutuhkannya.

---

# 95. Mass Assignment Protection

Backend tidak boleh melakukan:

```text
UPDATE table
SET everything_from_request
```

secara langsung.

Setiap endpoint memiliki field allowlist.

Contoh Petugas tidak boleh mengirim:

```json
{
  "status": "APPROVED"
}
```

ke Payment/Recap update endpoint dan berharap backend menerimanya.

Workflow status hanya berubah melalui endpoint action yang resmi.

---

# 96. Workflow Action Endpoint

Status tidak diubah menggunakan generic:

```text
PATCH /location-recaps/{id}
{
  "status": "APPROVED"
}
```

Hal ini tidak diperbolehkan.

Gunakan action-specific endpoint:

```text
POST /location-recaps/{id}/submit

POST /location-recaps/{id}/request-revision

POST /location-recaps/{id}/approve

POST /location-recaps/{id}/request-correction
```

Dengan demikian intent dan authorization lebih jelas.

---

# 97. API Audit Integration

Mutation penting menghasilkan Audit Log.

Contoh:

```text
POST /payments
→ CREATE_PAYMENT

PATCH /payments/{id}
→ UPDATE_PAYMENT

POST /payments/{id}/cancel
→ CANCEL_PAYMENT

POST /location-recaps/{id}/submit
→ SUBMIT_RECAP

POST /location-recaps/{id}/approve
→ APPROVE_RECAP
```

Audit failure pada transaction kritis harus dipertimbangkan sesuai transaction design.

---

# 98. Offline Evolution

V1 tetap:

```text
ONLINE-FIRST
```

Namun API sudah dipersiapkan untuk future offline write melalui:

* UUIDv7 client-generated IDs;
* idempotent create operation;
* stable timestamps;
* conflict responses;
* versioned API;
* repository abstraction.

---

# 99. Future Offline Payment

Future flow:

```text
Flutter Offline
      ↓
Generate Payment UUIDv7
      ↓
Save Local
      ↓
PENDING_SYNC
      ↓
Network returns
      ↓
POST /payments
      ↓
Server Validation
      ↓
SYNCED
```

API create Payment yang sama tetap dapat digunakan.

---

# 100. Workflow Tetap Online

Walaupun offline Payment nanti dikembangkan, endpoint berikut tetap online-only:

```text
/submit
/request-revision
/approve
/request-correction
/close
```

karena membutuhkan current server state.

---

# 101. OpenAPI Contract

Selain dokumen ini, kontrak API saat implementasi sebaiknya dituangkan dalam:

```text
OpenAPI
```

Contoh lokasi:

```text
docs/api/openapi.yaml
```

OpenAPI digunakan untuk mendokumentasikan secara machine-readable:

* endpoint;
* schema;
* parameter;
* response;
* authentication;
* error.

Dokumen `14-api-design.md` tetap menjelaskan alasan dan keputusan desain.

---

# 102. Development API Documentation

Pada environment development dapat disediakan API documentation UI yang membaca OpenAPI specification.

Documentation production tidak perlu dibuka secara publik apabila tidak diperlukan.

---

# 103. Endpoint Summary — Authentication

| Method | Endpoint        | Access          |
| ------ | --------------- | --------------- |
| POST   | `/auth/login`   | Public          |
| POST   | `/auth/refresh` | Refresh Session |
| POST   | `/auth/logout`  | Authenticated   |
| GET    | `/auth/me`      | Authenticated   |

---

# 104. Endpoint Summary — Users

| Method | Endpoint                               | Access     |
| ------ | -------------------------------------- | ---------- |
| GET    | `/users`                               | ADMIN_DESA |
| POST   | `/users`                               | ADMIN_DESA |
| GET    | `/users/{userId}`                      | ADMIN_DESA |
| PATCH  | `/users/{userId}`                      | ADMIN_DESA |
| POST   | `/users/{userId}/location-assignments` | ADMIN_DESA |
| PATCH  | `/location-assignments/{assignmentId}` | ADMIN_DESA |

---

# 105. Endpoint Summary — Locations

| Method | Endpoint                  | Access                     |
| ------ | ------------------------- | -------------------------- |
| GET    | `/locations`              | Authenticated sesuai scope |
| POST   | `/locations`              | ADMIN_DESA                 |
| GET    | `/locations/{locationId}` | Authenticated sesuai scope |
| PATCH  | `/locations/{locationId}` | ADMIN_DESA                 |

---

# 106. Endpoint Summary — Period

| Method | Endpoint                             | Access        |
| ------ | ------------------------------------ | ------------- |
| GET    | `/zakat-periods`                     | Authenticated |
| GET    | `/zakat-periods/active`              | Authenticated |
| POST   | `/zakat-periods`                     | ADMIN_DESA    |
| POST   | `/zakat-periods/{periodId}/activate` | ADMIN_DESA    |
| POST   | `/zakat-periods/{periodId}/close`    | ADMIN_DESA    |
| GET    | `/zakat-periods/{periodId}/rules`    | Authenticated |
| PUT    | `/zakat-periods/{periodId}/rules`    | ADMIN_DESA    |

---

# 107. Endpoint Summary — Families

| Method | Endpoint               | Access                         |
| ------ | ---------------------- | ------------------------------ |
| GET    | `/families`            | Authorized scope               |
| GET    | `/families/{familyId}` | Authorized scope               |
| PATCH  | `/families/{familyId}` | PETUGAS pada editable workflow |

Normal Family creation terjadi melalui Payment flow.

---

# 108. Endpoint Summary — Payments

| Method | Endpoint                       | Access                         |
| ------ | ------------------------------ | ------------------------------ |
| GET    | `/payments`                    | Authorized scope               |
| POST   | `/payments`                    | PETUGAS                        |
| GET    | `/payments/{paymentId}`        | Authorized scope               |
| PATCH  | `/payments/{paymentId}`        | PETUGAS pada editable workflow |
| POST   | `/payments/{paymentId}/cancel` | PETUGAS pada editable workflow |

Admin tidak memiliki normal direct Payment mutation.

---

# 109. Endpoint Summary — Recaps

| Method | Endpoint                                           | Access       |
| ------ | -------------------------------------------------- | ------------ |
| GET    | `/location-recaps`                                 | Role + scope |
| GET    | `/location-recaps/{recapId}`                       | Role + scope |
| POST   | `/location-recaps/{recapId}/submit`                | PETUGAS      |
| POST   | `/location-recaps/{recapId}/request-revision`      | ADMIN_DESA   |
| POST   | `/location-recaps/{recapId}/approve`               | ADMIN_DESA   |
| POST   | `/location-recaps/{recapId}/request-correction`    | ADMIN_DESA   |
| GET    | `/location-recaps/{recapId}/versions`              | Role + scope |
| GET    | `/location-recaps/{recapId}/versions/{revisionNo}` | Role + scope |
| GET    | `/location-recaps/{recapId}/workflow-events`       | Role + scope |

---

# 110. Endpoint Summary — Dashboard and Reports

| Method | Endpoint                         | Access           |
| ------ | -------------------------------- | ---------------- |
| GET    | `/dashboard`                     | Authenticated    |
| GET    | `/reports/location-recap`        | Role + scope     |
| GET    | `/reports/village-recap`         | ADMIN_DESA / DKM |
| GET    | `/reports/location-recap/export` | Role + scope     |
| GET    | `/reports/village-recap/export`  | ADMIN_DESA / DKM |

---

# 111. Endpoint Summary — Audit

| Method | Endpoint                | Access     |
| ------ | ----------------------- | ---------- |
| GET    | `/audit-logs`           | ADMIN_DESA |
| GET    | `/audit-logs/{auditId}` | ADMIN_DESA |

---

# 112. Authorization Matrix

| Operation              |             PETUGAS             | ADMIN_DESA | DKM |
| ---------------------- | :-----------------------------: | :--------: | :-: |
| Login                  |                ✅                |      ✅     |  ✅  |
| Create Payment         |          ✅ own location         |      ❌     |  ❌  |
| Update Payment         | ✅ own location + editable state |      ❌     |  ❌  |
| Cancel Payment         | ✅ own location + editable state |      ❌     |  ❌  |
| Submit Recap           |          ✅ own location         |      ❌     |  ❌  |
| Request Revision       |                ❌                |      ✅     |  ❌  |
| Approve Recap          |                ❌                |      ✅     |  ❌  |
| Request Correction     |                ❌                |      ✅     |  ❌  |
| Manage Users           |                ❌                |      ✅     |  ❌  |
| Manage Locations       |                ❌                |      ✅     |  ❌  |
| Manage Period          |                ❌                |      ✅     |  ❌  |
| View Village Recap     |         sesuai kebutuhan        |      ✅     |  ✅  |
| Export Official Report |         sesuai kebutuhan        |      ✅     |  ✅  |
| View Audit             |                ❌                |      ✅     |  ❌  |

---

# 113. Resource Scope Rule

Authorization tidak hanya berdasarkan role.

Contoh Petugas:

```text
User Role:
PETUGAS

Assigned Location:
Location A
```

Request:

```text
GET /payments/{paymentFromLocationB}
```

harus menghasilkan:

```text
403 Forbidden
```

meskipun role `PETUGAS` secara umum boleh melihat Payment.

---

# 114. Response Data Minimization

API hanya mengembalikan data yang diperlukan.

Contoh list Payment tidak perlu selalu mengirim:

* full audit history;
* refresh session;
* user security fields;
* seluruh recap versions.

Detail tambahan diperoleh melalui endpoint khusus apabila diperlukan.

---

# 115. API Performance Principles

API harus menghindari:

```text
N+1 query
unbounded list
full table scan
unnecessary large response
```

Gunakan:

* indexed filter;
* pagination;
* selected fields;
* efficient joins;
* aggregate query yang sesuai.

---

# 116. Report Performance

Report dapat membutuhkan query lebih berat dibanding operasi normal.

Report generation harus:

* menggunakan query teroptimasi;
* tidak memodifikasi data;
* memiliki timeout yang sesuai;
* menghindari loading seluruh database ke memory jika tidak diperlukan.

---

# 117. API Health Endpoint

Deployment dapat menyediakan:

```text
GET /health
```

Endpoint ini tidak menggunakan `/api/v1` jika diperlakukan sebagai infrastructure endpoint.

Contoh:

```json
{
  "status": "ok"
}
```

Health endpoint tidak boleh membocorkan:

* database password;
* internal configuration;
* secrets.

---

# 118. Backward Compatibility

Perubahan non-breaking dapat dilakukan dalam V1 seperti:

* menambah optional field;
* menambah endpoint;
* menambah error code.

Perubahan breaking seperti:

* menghapus field;
* mengubah tipe field;
* mengubah makna endpoint secara fundamental;

harus mempertimbangkan version baru atau migration strategy untuk client.

---

# 119. Keputusan Final API V1

Keputusan berikut dianggap baseline resmi.

## API Style

REST API melalui HTTPS.

## Version

```text
/api/v1
```

## JSON Naming

```text
camelCase
```

## Database Naming

```text
snake_case
```

## Authentication

Bearer short-lived access token + refresh session.

## Authorization

Role + Resource Scope + Business State.

## Payment Creation

Petugas langsung mengirim form Payment.

Backend melakukan automatic Family candidate detection.

## Family Candidate

Jika kandidat ditemukan:

```text
409 FAMILY_CONFIRMATION_REQUIRED
```

dan Petugas memberikan konfirmasi.

## Family Merge

Tidak pernah auto-merge ketika belum pasti.

## Payment Identifier

Client menghasilkan UUIDv7 sebelum create request.

## Idempotency

UUIDv7 dan optional `Idempotency-Key` digunakan untuk mengurangi duplicate write.

## Payment Cancellation

Menggunakan action endpoint:

```text
POST /payments/{id}/cancel
```

bukan `DELETE`.

## Workflow

Status Recap berubah hanya melalui action-specific endpoint.

## Admin Override

Admin tidak memiliki direct Payment mutation pada V1.

## Correction

```text
APPROVED
→ CORRECTION_REQUIRED
→ SUBMITTED
→ APPROVED
```

## Pagination

Cursor-based pagination.

## Reporting

Report dihitung oleh backend.

## Audit

Mutation penting menghasilkan audit event.

## Offline

V1 online-first tetapi Payment API disiapkan agar dapat digunakan future offline synchronization.

---

# 120. Hal yang Akan Diperdalam pada Security Design

Beberapa detail sengaja belum dikunci pada API Design karena akan dibahas di `15-security-design.md`, antara lain:

* access token expiration;
* refresh token expiration;
* refresh token rotation;
* password policy;
* Argon2id configuration;
* confirmation token implementation;
* exact rate limit;
* brute-force protection;
* device/session management;
* token revocation strategy;
* authorization middleware strategy;
* secure error logging;
* production security headers;
* threat model.

---

# 121. Kesimpulan

API V1 menjadi boundary utama antara Flutter dan backend.

Arsitektur komunikasi:

```text
Flutter + Riverpod
       ↓
     HTTPS
       ↓
REST API /api/v1
       ↓
Authentication
       ↓
Authorization
       ↓
Validation
       ↓
Application Use Case
       ↓
Domain Rules
       ↓
Drizzle ORM
       ↓
MySQL
```

Payment flow utama:

```text
Petugas Fill Form
       ↓
POST /payments
       ↓
Family Candidate Check
       ↓
┌───────────────────┬────────────────────────┐
No Candidate        Candidate Found
       │                     │
       ▼                     ▼
Create Family       Confirmation Required
Registration                 │
Payment                      ▼
       │             Existing / New Family
       └───────────────┬─────┘
                       ▼
                  Create Payment
```

Recap workflow:

```text
DRAFT
  ↓
SUBMITTED
  ├── REVISION_REQUIRED
  │          ↓
  │      SUBMITTED
  │
  └── APPROVED
          ↓
   CORRECTION_REQUIRED
          ↓
      SUBMITTED
          ↓
       APPROVED
```

API menggunakan UUIDv7, server-side validation, role + resource scope authorization, cursor pagination, idempotent Payment creation, workflow-specific endpoint, standardized errors, dan audit integration.

Dokumen ini menjadi **baseline final API Design V1**, tetapi tetap dapat dikembangkan secara terkontrol jika requirement atau architecture berubah.

Tahap berikutnya:

`15-security-design.md`
