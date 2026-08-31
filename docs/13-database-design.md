# Database Design

## 1. Tujuan Dokumen

Dokumen ini mendefinisikan desain database untuk Sistem Pendataan dan Rekapitulasi Zakat Fitrah Desa V1.

Database Design mencakup:

* struktur tabel;
* primary key;
* foreign key;
* relationship;
* cardinality;
* constraint;
* index;
* data historis;
* family registration;
* payment history;
* recap workflow;
* recap revision;
* authentication session;
* audit log;
* transaction;
* data integrity;
* scalability;
* extensibility.

Baseline teknologi:

```text id="db01"
Database       : MySQL
ORM            : Drizzle ORM
Driver         : mysql2
Identifier     : UUIDv7
Naming Style   : snake_case
```

---

# 2. Status Database Design

Dokumen ini menjadi **baseline final untuk V1**.

Final tidak berarti struktur database tidak boleh berubah selamanya.

Database dapat dikembangkan apabila:

* ditemukan requirement baru;
* ditemukan kebutuhan operasional baru;
* implementasi membutuhkan struktur tambahan;
* performa membutuhkan index baru;
* offline synchronization dikembangkan;
* module baru ditambahkan;
* kebutuhan security berubah.

Setiap perubahan yang signifikan harus melalui:

```text id="db02"
Requirement / Design Change
        ↓
Update Database Design
        ↓
Database Migration
        ↓
Update API / Application
        ↓
Testing
        ↓
Update change-log.md
```

Perubahan struktur database production tidak dilakukan secara langsung tanpa migration.

---

# 3. Tujuan Desain Database

Database harus mampu:

1. Menyimpan data keluarga secara konsisten.
2. Memisahkan identitas keluarga dari keikutsertaan zakat per periode.
3. Menyimpan histori pembayaran.
4. Mendukung lebih dari satu pembayaran untuk satu keluarga apabila diperlukan.
5. Memisahkan data antar periode zakat.
6. Mendukung authorization berdasarkan lokasi.
7. Menghitung rekap secara otomatis dari data sumber.
8. Menyimpan histori submission dan approval.
9. Menyimpan histori correction setelah approval.
10. Mendukung audit.
11. Mendukung query yang efisien.
12. Mendukung pengembangan offline synchronization di masa depan.
13. Mudah dipahami dan dipelihara.

---

# 4. Konvensi Penamaan

Nama teknis database menggunakan Bahasa Inggris.

Nama tabel dan atribut menggunakan:

```text id="db03"
snake_case
```

Contoh:

```text id="db04"
zakat_registrations
location_recaps
normalized_head_name
created_at
```

Primary key setiap entity menggunakan nama:

```text id="db05"
id
```

Contoh:

```text id="db06"
users.id
families.id
payments.id
locations.id
```

Foreign key menggunakan nama entity tujuan.

Contoh:

```text id="db07"
user_id
family_id
period_id
location_id
registration_id
recap_id
```

---

# 5. Strategi Identifier

Primary key entity utama menggunakan:

```text id="db08"
UUIDv7
```

Contoh:

```text id="db09"
families.id
```

Kolom `id` merupakan:

```text id="db10"
PRIMARY KEY
+
UUIDv7 value
```

UUID bukan atribut tambahan.

Sistem tidak menggunakan:

```text id="db11"
id = AUTO_INCREMENT
uuid = UUID
```

untuk entity yang sama.

Cukup satu identifier:

```text id="db12"
id
```

dengan nilai UUIDv7.

---

# 6. Penyimpanan UUID di MySQL

Target production menggunakan:

```text id="db13"
BINARY(16)
```

untuk menyimpan UUID.

Contoh:

```text id="db14"
families.id
BINARY(16)
PRIMARY KEY
```

Pada application layer, UUID dapat direpresentasikan sebagai string.

Konversi UUID string dan `BINARY(16)` dilakukan pada data access layer.

Penggunaan `BINARY(16)` dipilih karena:

* storage lebih kecil;
* index lebih kecil;
* comparison lebih efisien;
* cocok dengan karakter UUIDv7 yang time-ordered.

---

# 7. Model Data Utama

Model utama:

```text id="db15"
FAMILY
   │
   ▼
ZAKAT_REGISTRATION
   │
   ▼
PAYMENT
```

Artinya:

### `families`

Menyimpan identitas keluarga tingkat desa.

### `zakat_registrations`

Menyimpan keikutsertaan keluarga pada satu periode zakat.

### `payments`

Menyimpan kejadian pembayaran zakat.

Contoh:

```text id="db16"
Family Ahmad
│
├── Registration 2026
│      └── Payment #1
│
└── Registration 2027
       └── Payment #1
```

Jika terdapat pembayaran tambahan:

```text id="db17"
Family Ahmad
│
└── Registration 2027
       ├── Payment #1
       └── Payment #2
```

---

# 8. Alasan Memisahkan Family dan Registration

Tidak disarankan membuat:

```text id="db18"
Ahmad 2026
Ahmad 2027
Ahmad 2028
```

sebagai family yang berbeda.

Lebih baik:

```text id="db19"
Family Ahmad
   │
   ├── Registration 2026
   ├── Registration 2027
   └── Registration 2028
```

Dengan demikian:

```text id="db20"
Family
```

menjadi identitas jangka panjang.

Sedangkan:

```text id="db21"
Zakat Registration
```

menjadi konteks keikutsertaan per periode.

---

# 9. Daftar Tabel

| Table                       | Fungsi                                |
| --------------------------- | ------------------------------------- |
| `users`                     | Menyimpan pengguna aplikasi           |
| `locations`                 | Menyimpan lokasi penerimaan zakat     |
| `user_location_assignments` | Menghubungkan Petugas dengan lokasi   |
| `zakat_periods`             | Menyimpan periode zakat               |
| `zakat_period_rules`        | Menyimpan ketentuan zakat per periode |
| `families`                  | Master keluarga                       |
| `zakat_registrations`       | Keikutsertaan keluarga per periode    |
| `payments`                  | Riwayat pembayaran zakat              |
| `location_recaps`           | Current workflow state rekap lokasi   |
| `recap_versions`            | Snapshot rekap pada setiap submission |
| `recap_workflow_events`     | Riwayat perubahan status rekap        |
| `refresh_sessions`          | Refresh authentication session        |
| `audit_logs`                | Riwayat aktivitas penting             |

---

# 10. High-Level ERD

```text id="db22"
USERS
 │
 ├──< USER_LOCATION_ASSIGNMENTS >── LOCATIONS
 │                                      │
 │                                      ├──< PAYMENTS
 │                                      │
 │                                      └──< LOCATION_RECAPS
 │                                                │
 │                                                ├──< RECAP_VERSIONS
 │                                                │
 │                                                └──< RECAP_WORKFLOW_EVENTS
 │
 ├──< REFRESH_SESSIONS
 │
 └──< AUDIT_LOGS


FAMILIES
   │
   │ 1
   ▼
ZAKAT_REGISTRATIONS
   │
   │ 1:N
   ▼
PAYMENTS


ZAKAT_PERIODS
   │
   ├── 1:1 ZAKAT_PERIOD_RULES
   ├──< ZAKAT_REGISTRATIONS
   └──< LOCATION_RECAPS
```

---

# 11. Table `users`

Tabel `users` menyimpan seluruh akun pengguna aplikasi.

| Column          | Type         | Constraint            |
| --------------- | ------------ | --------------------- |
| `id`            | BINARY(16)   | PK                    |
| `name`          | VARCHAR(150) | NOT NULL              |
| `username`      | VARCHAR(100) | NOT NULL, UNIQUE      |
| `email`         | VARCHAR(255) | NULL, UNIQUE          |
| `password_hash` | VARCHAR(255) | NOT NULL              |
| `role`          | ENUM         | NOT NULL              |
| `is_active`     | BOOLEAN      | NOT NULL DEFAULT TRUE |
| `created_at`    | DATETIME(3)  | NOT NULL              |
| `updated_at`    | DATETIME(3)  | NOT NULL              |

Nilai `role`:

```text id="db23"
PETUGAS
ADMIN_DESA
DKM
```

---

# 12. Aturan `users`

Password asli tidak boleh disimpan.

Database hanya menyimpan:

```text id="db24"
password_hash
```

User yang sudah memiliki histori aktivitas tidak dihapus secara normal.

Jika akun tidak digunakan:

```text id="db25"
is_active = false
```

---

# 13. Table `locations`

Tabel `locations` menyimpan lokasi penerimaan zakat.

| Column       | Type         | Constraint            |
| ------------ | ------------ | --------------------- |
| `id`         | BINARY(16)   | PK                    |
| `name`       | VARCHAR(150) | NOT NULL              |
| `type`       | ENUM         | NOT NULL              |
| `address`    | VARCHAR(255) | NULL                  |
| `is_active`  | BOOLEAN      | NOT NULL DEFAULT TRUE |
| `created_at` | DATETIME(3)  | NOT NULL              |
| `updated_at` | DATETIME(3)  | NOT NULL              |

Nilai `type`:

```text id="db26"
MASJID
MUSHOLLA
SD
TK
```

---

# 14. Lifecycle Lokasi

Lokasi yang sudah memiliki histori pembayaran tidak dihapus permanen.

Jika lokasi tidak digunakan:

```text id="db27"
is_active = false
```

Lokasi tetap tersedia untuk:

* historical report;
* audit;
* recap periode sebelumnya.

Tetapi tidak dapat digunakan untuk transaksi baru.

---

# 15. Table `user_location_assignments`

Tabel ini menghubungkan Petugas dengan lokasi tugasnya.

| Column        | Type        | Constraint            |
| ------------- | ----------- | --------------------- |
| `id`          | BINARY(16)  | PK                    |
| `user_id`     | BINARY(16)  | FK → `users.id`       |
| `location_id` | BINARY(16)  | FK → `locations.id`   |
| `is_active`   | BOOLEAN     | NOT NULL DEFAULT TRUE |
| `created_at`  | DATETIME(3) | NOT NULL              |
| `updated_at`  | DATETIME(3) | NOT NULL              |

Constraint:

```text id="db28"
UNIQUE(user_id, location_id)
```

---

# 16. Alasan Menggunakan Assignment Table

Tidak menyimpan `location_id` secara permanen langsung pada `users`.

Dengan assignment table, sistem nantinya dapat mendukung:

```text id="db29"
Petugas A
├── Location A
└── Location B
```

tanpa perubahan struktur besar.

V1 tetap dapat menerapkan business rule bahwa seorang Petugas hanya memiliki satu lokasi aktif apabila diperlukan.

---

# 17. Table `zakat_periods`

Tabel ini menyimpan periode zakat.

| Column       | Type              | Constraint       |
| ------------ | ----------------- | ---------------- |
| `id`         | BINARY(16)        | PK               |
| `year`       | SMALLINT UNSIGNED | NOT NULL, UNIQUE |
| `start_date` | DATE              | NOT NULL         |
| `end_date`   | DATE              | NOT NULL         |
| `status`     | ENUM              | NOT NULL         |
| `created_by` | BINARY(16)        | FK → `users.id`  |
| `created_at` | DATETIME(3)       | NOT NULL         |
| `updated_at` | DATETIME(3)       | NOT NULL         |

Nilai `status`:

```text id="db30"
DRAFT
ACTIVE
CLOSED
```

---

# 18. Aturan Periode Aktif

Secara normal hanya boleh terdapat satu periode:

```text id="db31"
ACTIVE
```

Backend melakukan pengecekan dalam transaction sebelum mengaktifkan periode.

```text id="db32"
Check current ACTIVE period
        ↓
Tidak ada
        ↓
Activate selected period
```

---

# 19. Table `zakat_period_rules`

Menyimpan ketentuan zakat pada satu periode.

| Column                    | Type            | Constraint                  |
| ------------------------- | --------------- | --------------------------- |
| `period_id`               | BINARY(16)      | PK, FK → `zakat_periods.id` |
| `rice_kg_per_person`      | DECIMAL(10,3)   | NULL                        |
| `money_amount_per_person` | BIGINT UNSIGNED | NULL                        |
| `notes`                   | TEXT            | NULL                        |
| `updated_by`              | BINARY(16)      | FK → `users.id`             |
| `created_at`              | DATETIME(3)     | NOT NULL                    |
| `updated_at`              | DATETIME(3)     | NOT NULL                    |

Relasi:

```text id="db33"
zakat_periods
      │
      │ 1:1
      ▼
zakat_period_rules
```

---

# 20. Data Type Nominal Uang

Nominal Rupiah tidak menggunakan:

```text id="db34"
FLOAT
DOUBLE
```

Gunakan:

```text id="db35"
BIGINT UNSIGNED
```

Contoh:

```text id="db36"
Rp50.000
```

disimpan sebagai:

```text id="db37"
50000
```

Hal ini menghindari floating-point rounding error.

---

# 21. Data Type Berat Beras

Berat beras menggunakan:

```text id="db38"
DECIMAL(10,3)
```

Contoh:

```text id="db39"
2.500
5.000
7.500
```

Satuan yang digunakan adalah kilogram.

---

# 22. Table `families`

Tabel `families` menyimpan master keluarga tingkat desa.

| Column                 | Type         | Constraint      |
| ---------------------- | ------------ | --------------- |
| `id`                   | BINARY(16)   | PK              |
| `head_name`            | VARCHAR(150) | NOT NULL        |
| `normalized_head_name` | VARCHAR(150) | NOT NULL        |
| `hamlet`               | VARCHAR(100) | NULL            |
| `rt`                   | VARCHAR(10)  | NULL            |
| `rw`                   | VARCHAR(10)  | NULL            |
| `address_detail`       | VARCHAR(255) | NULL            |
| `created_by`           | BINARY(16)   | FK → `users.id` |
| `created_at`           | DATETIME(3)  | NOT NULL        |
| `updated_at`           | DATETIME(3)  | NOT NULL        |

---

# 23. Identifier Family

Identitas teknis Family adalah:

```text id="db40"
families.id
```

dengan UUIDv7.

Atribut seperti:

```text id="db41"
head_name
hamlet
rt
rw
```

bukan primary key.

---

# 24. Alasan `rt` dan `rw` Menggunakan VARCHAR

`rt` dan `rw` tidak menggunakan integer karena nilai seperti:

```text id="db42"
01
002
```

harus mempertahankan leading zero.

Penggunaan `VARCHAR` juga lebih fleksibel apabila format administrasi wilayah berubah.

---

# 25. `normalized_head_name`

Digunakan untuk membantu pencarian kandidat keluarga.

Contoh:

```text id="db43"
head_name:
"  Ahmad  "

normalized_head_name:
"ahmad"
```

Normalisasi dapat mencakup:

* trim;
* lowercase;
* menghapus whitespace ganda.

`head_name` asli tetap dipertahankan untuk display.

---

# 26. Duplicate Detection Family

Index awal:

```text id="db44"
INDEX(
    normalized_head_name,
    hamlet,
    rt,
    rw
)
```

Index ini **tidak UNIQUE**.

Artinya database tetap memperbolehkan dua Family berbeda memiliki:

```text id="db45"
Ahmad
Dusun Manis
RT 02
RW 01
```

yang sama.

UUIDv7 tetap menjadi pembeda teknis.

---

# 27. Flow Pencarian Family

Petugas tidak diwajibkan melakukan pencarian manual sebelum input.

Workflow:

```text id="db46"
Fill Payment Form
       ↓
Save
       ↓
Backend normalize family data
       ↓
Indexed candidate search
```

Jika tidak ditemukan kandidat:

```text id="db47"
Create Family
+
Create Registration
+
Create Payment
```

Jika ditemukan kandidat:

```text id="db48"
Return candidate
       ↓
Petugas confirmation
```

Sistem tidak melakukan auto-merge jika Family belum dapat dipastikan sama.

---

# 28. Table `zakat_registrations`

Tabel ini menyimpan keikutsertaan sebuah Family dalam satu periode zakat.

| Column       | Type        | Constraint              |
| ------------ | ----------- | ----------------------- |
| `id`         | BINARY(16)  | PK                      |
| `family_id`  | BINARY(16)  | FK → `families.id`      |
| `period_id`  | BINARY(16)  | FK → `zakat_periods.id` |
| `created_by` | BINARY(16)  | FK → `users.id`         |
| `created_at` | DATETIME(3) | NOT NULL                |
| `updated_at` | DATETIME(3) | NOT NULL                |

Constraint:

```text id="db49"
UNIQUE(family_id, period_id)
```

---

# 29. Aturan Registration

Satu Family hanya memiliki satu Registration pada satu Period.

Valid:

```text id="db50"
Family Ahmad
├── Registration 2026
└── Registration 2027
```

Tidak valid:

```text id="db51"
Family Ahmad
├── Registration 2026 A
└── Registration 2026 B
```

---

# 30. Kenapa Location Tidak Disimpan di Registration?

Lokasi disimpan pada Payment.

Hal ini karena Payment merepresentasikan tempat transaksi sebenarnya.

Contoh kasus pengecualian:

```text id="db52"
Registration Ahmad 2027
├── Payment #1 → Musholla A
└── Payment #2 → Masjid B
```

Walaupun kemungkinan kasus ini kecil, struktur database tetap dapat menanganinya tanpa perubahan schema.

---

# 31. Table `payments`

Tabel ini menyimpan transaksi pembayaran zakat.

| Column            | Type              | Constraint                    |
| ----------------- | ----------------- | ----------------------------- |
| `id`              | BINARY(16)        | PK                            |
| `registration_id` | BINARY(16)        | FK → `zakat_registrations.id` |
| `location_id`     | BINARY(16)        | FK → `locations.id`           |
| `recorded_by`     | BINARY(16)        | FK → `users.id`               |
| `payment_date`    | DATE              | NOT NULL                      |
| `total_people`    | SMALLINT UNSIGNED | NOT NULL                      |
| `rice_people`     | SMALLINT UNSIGNED | NOT NULL DEFAULT 0            |
| `rice_kg`         | DECIMAL(10,3)     | NOT NULL DEFAULT 0            |
| `money_people`    | SMALLINT UNSIGNED | NOT NULL DEFAULT 0            |
| `money_amount`    | BIGINT UNSIGNED   | NOT NULL DEFAULT 0            |
| `status`          | ENUM              | NOT NULL DEFAULT ACTIVE       |
| `cancel_reason`   | VARCHAR(500)      | NULL                          |
| `cancelled_by`    | BINARY(16)        | NULL, FK → `users.id`         |
| `cancelled_at`    | DATETIME(3)       | NULL                          |
| `created_at`      | DATETIME(3)       | NOT NULL                      |
| `updated_at`      | DATETIME(3)       | NOT NULL                      |

Nilai `status`:

```text id="db53"
ACTIVE
CANCELLED
```

---

# 32. Validasi Payment

Rule utama:

```text id="db54"
total_people >= 1
```

Konsistensi metode pembayaran:

```text id="db55"
rice_people + money_people = total_people
```

Nilai tidak boleh negatif:

```text id="db56"
rice_people >= 0
money_people >= 0
rice_kg >= 0
money_amount >= 0
```

Minimal satu metode pembayaran harus digunakan.

---

# 33. Database CHECK Constraint

Rule sederhana juga dijaga pada database.

Contoh konseptual:

```text id="db57"
CHECK(total_people > 0)

CHECK(rice_people + money_people = total_people)

CHECK(rice_kg >= 0)

CHECK(money_amount >= 0)
```

Backend tetap melakukan validation terlebih dahulu.

Database menjadi lapisan perlindungan tambahan.

---

# 34. Mixed Payment

Sistem mendukung:

```text id="db58"
Rice Only
Money Only
Rice + Money
```

Contoh:

```text id="db59"
total_people = 5

rice_people = 3
rice_kg = 7.500

money_people = 2
money_amount = 100000
```

---

# 35. Multiple Payment

Satu Registration dapat mempunyai lebih dari satu Payment.

```text id="db60"
Registration Ahmad
├── Payment #1
└── Payment #2
```

Kasus ini diperlakukan sebagai exception/safety net.

Normal workflow tetap diharapkan satu pembayaran yang sudah dipersiapkan sebelumnya.

Payment tambahan tidak menimpa Payment lama.

---

# 36. Payment Cancellation

Payment tidak di-hard-delete.

Pembatalan menggunakan:

```text id="db61"
status = CANCELLED
cancel_reason = ...
cancelled_by = ...
cancelled_at = ...
```

Payment dengan status `CANCELLED`:

* tetap tersimpan sebagai histori;
* tidak masuk perhitungan rekap.

---

# 37. Index `payments`

Initial indexes:

```text id="db62"
INDEX(registration_id)

INDEX(location_id)

INDEX(payment_date)

INDEX(location_id, status)

INDEX(registration_id, status)
```

Index dapat disesuaikan setelah query profiling.

---

# 38. Table `location_recaps`

Tabel ini menyimpan current workflow state rekap untuk satu lokasi dan satu periode.

| Column        | Type         | Constraint              |
| ------------- | ------------ | ----------------------- |
| `id`          | BINARY(16)   | PK                      |
| `location_id` | BINARY(16)   | FK → `locations.id`     |
| `period_id`   | BINARY(16)   | FK → `zakat_periods.id` |
| `status`      | ENUM         | NOT NULL                |
| `revision_no` | INT UNSIGNED | NOT NULL DEFAULT 0      |
| `created_at`  | DATETIME(3)  | NOT NULL                |
| `updated_at`  | DATETIME(3)  | NOT NULL                |

Constraint:

```text id="db63"
UNIQUE(location_id, period_id)
```

---

# 39. Recap Status

Nilai `status`:

```text id="db64"
DRAFT
SUBMITTED
REVISION_REQUIRED
APPROVED
CORRECTION_REQUIRED
```

---

# 40. Source of Truth Rekap

`location_recaps` tidak menyimpan total manual sebagai source of truth.

Live total dihitung dari:

```text id="db65"
ACTIVE payments
```

berdasarkan:

```text id="db66"
location
+
period
```

Dengan demikian tidak terjadi kondisi:

```text id="db67"
stored recap total
≠
actual payment total
```

---

# 41. Perhitungan Location Recap

Untuk satu Location dan Period:

### Total KK

```text id="db68"
COUNT(DISTINCT registration_id)
```

### Total Jiwa

```text id="db69"
SUM(total_people)
```

### Jiwa Beras

```text id="db70"
SUM(rice_people)
```

### Total Beras

```text id="db71"
SUM(rice_kg)
```

### Jiwa Uang

```text id="db72"
SUM(money_people)
```

### Total Uang

```text id="db73"
SUM(money_amount)
```

Hanya Payment:

```text id="db74"
status = ACTIVE
```

yang dihitung.

---

# 42. Table `recap_versions`

Tabel ini menyimpan immutable snapshot setiap kali rekap dikirim.

| Column            | Type            | Constraint                |
| ----------------- | --------------- | ------------------------- |
| `id`              | BINARY(16)      | PK                        |
| `recap_id`        | BINARY(16)      | FK → `location_recaps.id` |
| `revision_no`     | INT UNSIGNED    | NOT NULL                  |
| `total_families`  | INT UNSIGNED    | NOT NULL                  |
| `total_people`    | INT UNSIGNED    | NOT NULL                  |
| `rice_people`     | INT UNSIGNED    | NOT NULL                  |
| `rice_kg`         | DECIMAL(12,3)   | NOT NULL                  |
| `money_people`    | INT UNSIGNED    | NOT NULL                  |
| `money_amount`    | BIGINT UNSIGNED | NOT NULL                  |
| `submitted_by`    | BINARY(16)      | FK → `users.id`           |
| `submitted_at`    | DATETIME(3)     | NOT NULL                  |
| `decision`        | ENUM            | NOT NULL                  |
| `decided_by`      | BINARY(16)      | NULL, FK → `users.id`     |
| `decided_at`      | DATETIME(3)     | NULL                      |
| `decision_reason` | VARCHAR(1000)   | NULL                      |
| `created_at`      | DATETIME(3)     | NOT NULL                  |

Constraint:

```text id="db75"
UNIQUE(recap_id, revision_no)
```

---

# 43. Recap Version Decision

Nilai `decision`:

```text id="db76"
PENDING
REVISION_REQUIRED
APPROVED
```

Snapshot angka pada `recap_versions` tidak diubah setelah submission.

Hanya decision metadata yang berubah sesuai hasil review.

---

# 44. Alasan Menggunakan Recap Version

Misalnya:

```text id="db77"
Revision 1
Total KK   = 100
Total Jiwa = 300
```

setelah revisi menjadi:

```text id="db78"
Revision 2
Total KK   = 101
Total Jiwa = 304
```

Sistem harus tetap mengetahui angka yang sebelumnya pernah dikirim.

Karena itu:

```text id="db79"
recap_versions
```

berfungsi sebagai historical snapshot.

---

# 45. Workflow Revision

Awal:

```text id="db80"
location_recaps
revision_no = 0
status = DRAFT
```

Petugas submit:

```text id="db81"
recap_versions
revision_no = 1
decision = PENDING
```

Current recap:

```text id="db82"
revision_no = 1
status = SUBMITTED
```

Jika Admin meminta revision:

```text id="db83"
Version 1
decision = REVISION_REQUIRED
```

Current recap:

```text id="db84"
status = REVISION_REQUIRED
```

Petugas memperbaiki dan submit ulang:

```text id="db85"
Version 2
decision = PENDING
```

Current recap:

```text id="db86"
revision_no = 2
status = SUBMITTED
```

Setelah approval:

```text id="db87"
Version 2
decision = APPROVED
```

Current recap:

```text id="db88"
status = APPROVED
```

---

# 46. Correction Setelah Approval

Misalnya:

```text id="db89"
Revision 2
decision = APPROVED
```

Kemudian ditemukan kesalahan.

Current recap berubah:

```text id="db90"
CORRECTION_REQUIRED
```

Revision 2 tetap:

```text id="db91"
APPROVED
```

karena secara historis revision tersebut memang pernah disetujui.

Setelah Petugas melakukan correction:

```text id="db92"
Revision 3
decision = PENDING
```

Setelah approval ulang:

```text id="db93"
Revision 3
decision = APPROVED
```

dan current recap kembali:

```text id="db94"
APPROVED
```

---

# 47. Table `recap_workflow_events`

Menyimpan histori perubahan workflow rekap.

| Column        | Type          | Constraint                |
| ------------- | ------------- | ------------------------- |
| `id`          | BINARY(16)    | PK                        |
| `recap_id`    | BINARY(16)    | FK → `location_recaps.id` |
| `revision_no` | INT UNSIGNED  | NOT NULL                  |
| `from_status` | ENUM          | NULL                      |
| `to_status`   | ENUM          | NOT NULL                  |
| `action`      | VARCHAR(100)  | NOT NULL                  |
| `reason`      | VARCHAR(1000) | NULL                      |
| `actor_id`    | BINARY(16)    | FK → `users.id`           |
| `created_at`  | DATETIME(3)   | NOT NULL                  |

Contoh nilai `action`:

```text id="db95"
SUBMIT
REQUEST_REVISION
RESUBMIT
APPROVE
REQUEST_CORRECTION
```

---

# 48. Perbedaan Recap Version dan Workflow Event

`recap_versions` menjawab:

> Angka apa yang dikirim?

Sedangkan:

`recap_workflow_events` menjawab:

> Apa yang terjadi pada rekap?

Contoh:

```text id="db96"
recap_versions
→ total KK, jiwa, beras, uang

recap_workflow_events
→ SUBMITTED → REVISION_REQUIRED
```

---

# 49. Village Recap

V1 tidak membutuhkan tabel:

```text id="db97"
village_recaps
```

sebagai source of truth.

Village recap dihitung dari data sumber.

Hanya Location dengan:

```text id="db98"
location_recaps.status = APPROVED
```

yang masuk ke official village recap.

---

# 50. Perhitungan KK Tingkat Desa

Karena secara teknis satu Family dapat memiliki Payment pada lebih dari satu Location, total KK desa tidak menggunakan:

```text id="db99"
SUM(location_total_families)
```

secara langsung.

Lebih aman menggunakan:

```text id="db100"
COUNT(DISTINCT registration_id)
```

dari Payment yang memenuhi syarat official recap.

Ini mencegah Family yang sama dihitung dua kali.

---

# 51. Table `refresh_sessions`

Tabel ini menyimpan refresh authentication session.

| Column         | Type         | Constraint      |
| -------------- | ------------ | --------------- |
| `id`           | BINARY(16)   | PK              |
| `user_id`      | BINARY(16)   | FK → `users.id` |
| `token_hash`   | VARCHAR(255) | NOT NULL        |
| `expires_at`   | DATETIME(3)  | NOT NULL        |
| `last_used_at` | DATETIME(3)  | NULL            |
| `revoked_at`   | DATETIME(3)  | NULL            |
| `created_at`   | DATETIME(3)  | NOT NULL        |

Raw refresh token tidak disimpan pada database.

Hanya:

```text id="db101"
token_hash
```

yang disimpan.

---

# 52. Validitas Refresh Session

Session dianggap aktif jika:

```text id="db102"
revoked_at IS NULL
AND
expires_at > current_time
```

Logout melakukan:

```text id="db103"
revoked_at = current_time
```

---

# 53. Table `audit_logs`

Menyimpan aktivitas penting pengguna dan sistem.

| Column          | Type         | Constraint            |
| --------------- | ------------ | --------------------- |
| `id`            | BINARY(16)   | PK                    |
| `actor_id`      | BINARY(16)   | NULL, FK → `users.id` |
| `action`        | VARCHAR(100) | NOT NULL              |
| `resource_type` | VARCHAR(100) | NOT NULL              |
| `resource_id`   | BINARY(16)   | NULL                  |
| `before_data`   | JSON         | NULL                  |
| `after_data`    | JSON         | NULL                  |
| `metadata`      | JSON         | NULL                  |
| `request_id`    | VARCHAR(100) | NULL                  |
| `created_at`    | DATETIME(3)  | NOT NULL              |

---

# 54. Contoh Audit

```text id="db104"
action:
UPDATE_PAYMENT

resource_type:
PAYMENT

before_data:
{
  "total_people": 2,
  "rice_kg": 5
}

after_data:
{
  "total_people": 3,
  "rice_kg": 7.5
}
```

---

# 55. Data Sensitif pada Audit

Audit log tidak boleh menyimpan:

```text id="db105"
password
access_token
refresh_token
database_password
application_secret
```

Audit hanya menyimpan data yang memang diperlukan untuk accountability.

---

# 56. Audit Policy

Audit menggunakan pendekatan:

```text id="db106"
append-oriented
```

Normal user tidak dapat:

* mengedit audit;
* menghapus audit.

Retention policy dapat dikembangkan kemudian jika diperlukan.

---

# 57. Timestamp Strategy

Timestamp sistem menggunakan UTC.

Contoh:

```text id="db107"
created_at
updated_at
submitted_at
decided_at
cancelled_at
```

Flutter melakukan konversi untuk tampilan local timezone.

---

# 58. Business Date

Tanggal pembayaran menggunakan:

```text id="db108"
DATE
```

Contoh:

```text id="db109"
payment_date
```

karena merepresentasikan tanggal operasional dan tidak membutuhkan timezone conversion.

---

# 59. Referential Integrity

Foreign key menjaga hubungan antar entity.

Contoh:

```text id="db110"
payments.registration_id
→ zakat_registrations.id

zakat_registrations.family_id
→ families.id

zakat_registrations.period_id
→ zakat_periods.id

payments.location_id
→ locations.id

location_recaps.location_id
→ locations.id

location_recaps.period_id
→ zakat_periods.id
```

Database harus mencegah orphan record.

---

# 60. Delete Policy

Hard delete dihindari untuk data historis.

### `users`

Gunakan:

```text id="db111"
is_active = false
```

### `locations`

Gunakan:

```text id="db112"
is_active = false
```

### `payments`

Gunakan:

```text id="db113"
status = CANCELLED
```

### `recap_versions`

Tidak dihapus secara normal.

### `recap_workflow_events`

Tidak dihapus secara normal.

### `audit_logs`

Tidak dihapus secara normal.

### `families`

Family yang sudah memiliki histori tidak dihapus secara normal.

---

# 61. Transaction Boundary

Operasi yang terdiri dari beberapa perubahan data dan harus konsisten menggunakan database transaction.

---

# 62. Transaction — Family Baru

Jika Family belum tersedia:

```text id="db114"
BEGIN

Create Family
Create Zakat Registration
Create Payment
Create Audit Log

COMMIT
```

Jika salah satu proses utama gagal:

```text id="db115"
ROLLBACK
```

---

# 63. Transaction — Family Existing

Jika Family sudah ada:

```text id="db116"
BEGIN

Find/Create Registration
Create Payment
Create Audit Log

COMMIT
```

Payment lama tidak ditimpa.

---

# 64. Transaction — Submit Recap

```text id="db117"
BEGIN

Validate current recap status
Calculate current totals
Increment revision_no
Create recap_versions snapshot
Update location_recaps → SUBMITTED
Create recap_workflow_events
Create audit_logs

COMMIT
```

---

# 65. Transaction — Approve Recap

```text id="db118"
BEGIN

Validate status = SUBMITTED
Validate current revision
Update recap_versions decision = APPROVED
Update location_recaps status = APPROVED
Create workflow event
Create audit log

COMMIT
```

---

# 66. Transaction — Correction

```text id="db119"
BEGIN

Validate status = APPROVED
Update location_recaps → CORRECTION_REQUIRED
Create workflow event
Create audit log

COMMIT
```

Approved Recap Version sebelumnya tetap disimpan.

---

# 67. Initial Index Strategy

## `users`

```text id="db120"
UNIQUE(username)
UNIQUE(email)
```

## `user_location_assignments`

```text id="db121"
UNIQUE(user_id, location_id)
INDEX(location_id)
```

## `families`

```text id="db122"
INDEX(normalized_head_name, hamlet, rt, rw)
```

## `zakat_registrations`

```text id="db123"
UNIQUE(family_id, period_id)
INDEX(period_id)
```

## `payments`

```text id="db124"
INDEX(registration_id)
INDEX(location_id)
INDEX(payment_date)
INDEX(location_id, status)
INDEX(registration_id, status)
```

## `location_recaps`

```text id="db125"
UNIQUE(location_id, period_id)
INDEX(period_id, status)
```

## `recap_versions`

```text id="db126"
UNIQUE(recap_id, revision_no)
```

## `recap_workflow_events`

```text id="db127"
INDEX(recap_id, created_at)
```

## `refresh_sessions`

```text id="db128"
INDEX(user_id)
INDEX(expires_at)
```

## `audit_logs`

```text id="db129"
INDEX(actor_id, created_at)
INDEX(resource_type, resource_id)
INDEX(request_id)
```

---

# 68. Hindari Over-Indexing

Tidak setiap column membutuhkan index.

Index yang terlalu banyak menyebabkan:

* insert lebih berat;
* update lebih berat;
* penggunaan storage meningkat.

Index dibuat berdasarkan kebutuhan nyata seperti:

```text id="db130"
WHERE
JOIN
ORDER BY
pagination
query profiling
```

---

# 69. Pagination

Data yang dapat berkembang harus menggunakan pagination.

Contoh:

* `users`;
* `families`;
* `payments`;
* `audit_logs`;
* `recap_workflow_events`.

V1 dapat menggunakan pagination sederhana.

Cursor-based pagination dapat dikembangkan apabila volume data meningkat.

---

# 70. Performance Family Search

Duplicate candidate search menggunakan indexed field:

```text id="db131"
normalized_head_name
hamlet
rt
rw
```

Sistem tidak melakukan expensive fuzzy search terhadap seluruh Family pada V1.

Dengan index yang tepat, proses duplicate checking tidak diharapkan menjadi bottleneck pada skala desa.

---

# 71. Human Attribute Bukan Unique Identifier

Tidak dibuat:

```text id="db132"
UNIQUE(
    head_name,
    hamlet,
    rt,
    rw
)
```

karena Family berbeda dapat memiliki data yang sama.

Technical uniqueness berasal dari:

```text id="db133"
families.id
```

dengan UUIDv7.

---

# 72. NIK dan Family Card Number

NIK dan nomor KK tidak diwajibkan pada V1.

Alasan:

* belum terdapat operational requirement yang jelas;
* menerapkan data minimization;
* mengurangi penyimpanan data sensitif.

Jika nantinya benar-benar diperlukan, column tersebut dapat ditambahkan melalui migration.

---

# 73. Reporting

Location report menggunakan:

```text id="db134"
Period
+
Location
+
ACTIVE Payments
```

Official village report menggunakan data dari Location yang current recap-nya:

```text id="db135"
APPROVED
```

Report tidak bergantung pada total yang diketik manual.

---

# 74. Report File Persistence

File PDF/Excel tidak perlu disimpan permanen di database V1.

Report dibuat:

```text id="db136"
on demand
```

Jika nanti diperlukan arsip file resmi, dapat ditambahkan tabel:

```text id="db137"
report_exports
```

melalui migration.

---

# 75. Database Security

Production menggunakan database account khusus.

Tidak menggunakan:

```text id="db138"
root
```

untuk normal application access.

Flutter tidak memiliki akses langsung ke MySQL.

Arsitektur:

```text id="db139"
Flutter
   ↓
Backend
   ↓
MySQL
```

---

# 76. Sensitive Column

Column seperti:

```text id="db140"
users.password_hash
refresh_sessions.token_hash
```

tidak boleh dikembalikan melalui normal API response.

---

# 77. Drizzle Schema Organization

Schema Drizzle mengikuti backend module.

Contoh:

```text id="db141"
backend/src/modules/
│
├── auth/
│   └── schema/
│       └── refresh-sessions.schema.ts
│
├── users/
│   └── schema/
│       ├── users.schema.ts
│       └── user-location-assignments.schema.ts
│
├── locations/
│   └── schema/
│       └── locations.schema.ts
│
├── periods/
│   └── schema/
│       ├── zakat-periods.schema.ts
│       └── zakat-period-rules.schema.ts
│
├── families/
│   └── schema/
│       ├── families.schema.ts
│       └── zakat-registrations.schema.ts
│
├── payments/
│   └── schema/
│       └── payments.schema.ts
│
├── recaps/
│   └── schema/
│       ├── location-recaps.schema.ts
│       ├── recap-versions.schema.ts
│       └── recap-workflow-events.schema.ts
│
└── audit/
    └── schema/
        └── audit-logs.schema.ts
```

---

# 78. Migration Strategy

Semua perubahan schema melalui migration.

Contoh:

```text id="db142"
0001_create_users
0002_create_locations
0003_create_user_location_assignments
0004_create_zakat_periods
0005_create_zakat_period_rules
0006_create_families
0007_create_zakat_registrations
0008_create_payments
0009_create_location_recaps
0010_create_recap_versions
0011_create_recap_workflow_events
0012_create_refresh_sessions
0013_create_audit_logs
```

Migration yang sudah digunakan di production tidak diedit sembarangan.

Perubahan berikutnya menggunakan migration baru.

---

# 79. Backup dan Recovery

Production database harus memiliki backup berkala.

Backup minimal mencakup:

* schema;
* user;
* location;
* family;
* registration;
* payment;
* recap;
* recap versions;
* workflow history;
* audit.

Restore procedure harus diuji.

Backup tanpa prosedur restore yang berhasil belum dianggap cukup.

---

# 80. Data Retention

Historical zakat data dipertahankan.

V1 tidak melakukan automatic purge terhadap:

* `families`;
* `zakat_registrations`;
* `payments`;
* `location_recaps`;
* `recap_versions`;
* `recap_workflow_events`;
* `audit_logs`.

Retention policy dapat ditentukan kemudian berdasarkan kebutuhan administratif.

---

# 81. Dukungan Future Offline

UUIDv7 memungkinkan Mobile membuat identifier sebelum data dikirim ke server.

Contoh future flow:

```text id="db143"
Create Payment Offline
       ↓
Generate UUIDv7
       ↓
Save Local
       ↓
PENDING_SYNC
       ↓
Internet Available
       ↓
Send Payment with same UUID
```

Server tidak perlu menghasilkan ID baru.

---

# 82. Idempotency

API dapat menggunakan:

```text id="db144"
payment.id
```

atau:

```text id="db145"
Idempotency-Key
```

untuk mengenali request yang dikirim ulang.

Contoh:

```text id="db146"
Request pertama berhasil
↓
Response gagal diterima device
↓
Device retry
↓
Server mengenali operation yang sama
↓
Tidak membuat Payment kedua
```

Ini berguna untuk:

* double tap;
* unstable connection;
* future offline synchronization.

---

# 83. Data Integrity Layers

Data integrity tidak hanya bergantung pada database.

```text id="db147"
Flutter Validation
       ↓
Backend Validation
       ↓
Business Rules
       ↓
Database Constraints
       ↓
Foreign Keys
       ↓
Transactions
       ↓
Audit Trail
```

---

# 84. Cardinality

| Relationship                        | Cardinality       |
| ----------------------------------- | ----------------- |
| User → User Location Assignment     | 1:N               |
| Location → User Location Assignment | 1:N               |
| Family → Zakat Registration         | 1:N               |
| Zakat Period → Zakat Registration   | 1:N               |
| Zakat Registration → Payment        | 1:N               |
| Location → Payment                  | 1:N               |
| Location → Location Recap           | 1:N antar periode |
| Zakat Period → Location Recap       | 1:N               |
| Location Recap → Recap Version      | 1:N               |
| Location Recap → Workflow Event     | 1:N               |
| User → Refresh Session              | 1:N               |
| User → Audit Log                    | 1:N               |

---

# 85. Final Relationship Overview

```text id="db148"
FAMILIES
   │
   ▼
ZAKAT_REGISTRATIONS
   │
   ▼
PAYMENTS
```

Operational:

```text id="db149"
LOCATIONS
   │
   ├── PAYMENTS
   │
   └── LOCATION_RECAPS
            │
            ├── RECAP_VERSIONS
            └── RECAP_WORKFLOW_EVENTS
```

Security:

```text id="db150"
USERS
 │
 ├── USER_LOCATION_ASSIGNMENTS
 ├── REFRESH_SESSIONS
 └── AUDIT_LOGS
```

Period:

```text id="db151"
ZAKAT_PERIODS
 │
 ├── ZAKAT_PERIOD_RULES
 ├── ZAKAT_REGISTRATIONS
 └── LOCATION_RECAPS
```

---

# 86. Keputusan Final Database V1

Keputusan berikut menjadi baseline resmi V1.

## Identifier

Entity utama menggunakan UUIDv7.

## Primary Key

Setiap tabel menggunakan:

```text id="db152"
id
```

sebagai primary key, kecuali tabel one-to-one seperti `zakat_period_rules` yang dapat menggunakan `period_id` sebagai PK sekaligus FK.

## Family

`families` merupakan master Family tingkat desa.

## Registration

`zakat_registrations` menghubungkan Family dengan Zakat Period.

## Registration Uniqueness

```text id="db153"
UNIQUE(family_id, period_id)
```

## Payment

Satu Registration dapat memiliki satu atau lebih Payment.

## Payment Location

Lokasi transaksi disimpan pada `payments.location_id`.

## Payment Cancellation

Menggunakan status:

```text id="db154"
CANCELLED
```

bukan hard delete.

## Family Matching

Data manusia digunakan untuk candidate search tetapi bukan unique identifier.

## Recap

Satu Location hanya memiliki satu current recap pada satu Period.

## Live Recap Total

Dihitung dari Payment source data.

## Recap History

Setiap submission menghasilkan `recap_versions`.

## Workflow History

Perubahan status disimpan pada `recap_workflow_events`.

## Correction

Approved Recap Version lama tetap dipertahankan.

## Village Recap

Dihitung secara dinamis dan belum memerlukan tabel khusus pada V1.

## Authentication Session

Refresh token disimpan dalam bentuk hash.

## Audit

Audit menggunakan append-oriented log.

---

# 87. Hal yang Dapat Dikembangkan Kemudian

Database ini dirancang agar dapat dikembangkan untuk:

```text id="db155"
Offline synchronization
Multiple assigned locations
Zakat Mal
Mustahik management
Zakat distribution
Report archive
Device/session management
Enhanced family matching
Additional location types
Advanced analytics
```

Penambahan tersebut dilakukan melalui migration dan update design documentation.

---

# 88. Kesimpulan

Database V1 menggunakan struktur utama:

```text id="db156"
Family
  ↓
Zakat Registration
  ↓
Payment
```

Struktur tersebut memisahkan:

* identitas Family;
* participation per period;
* actual payment event.

Workflow rekap menggunakan:

```text id="db157"
Location Recap
      ↓
Recap Version
      ↓
Workflow Event
```

Sehingga sistem dapat mempertahankan:

* current workflow state;
* submitted recap history;
* revision history;
* correction history.

UUIDv7 digunakan sebagai nilai primary key untuk entity utama.

Database menggunakan MySQL dengan Drizzle ORM dan dirancang agar:

* relational;
* transaction-safe;
* auditable;
* scalable untuk kebutuhan sistem;
* mendukung historical data;
* dapat dikembangkan tanpa rewrite besar.

Dokumen ini merupakan **baseline final Database Design untuk V1**, tetapi tetap dapat dikembangkan melalui migration dan perubahan dokumentasi yang terkontrol.

Tahap desain selanjutnya adalah:

`14-api-design.md`
