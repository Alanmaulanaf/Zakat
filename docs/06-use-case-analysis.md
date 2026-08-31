# Use Case Analysis

## Actors

### Petugas Lokasi

Petugas yang bertugas mencatat zakat pada:

- Masjid
- Musholla
- SD
- TK

### Admin Desa

Pengguna yang melakukan administrasi dan pengelolaan
data pada tingkat desa.

### DKM

Pengguna yang dapat melihat hasil rekapitulasi dan laporan.

## Business Actor

### Warga

Warga bukan pengguna langsung aplikasi.

Warga melakukan pembayaran kepada petugas, kemudian
petugas melakukan pencatatan pada sistem.

---

# Use Cases

## UC-001 — Login

Actor:

- Petugas Lokasi
- Admin Desa
- DKM

## UC-002 — Logout

Actor:

- Petugas Lokasi
- Admin Desa
- DKM

## UC-003 — Kelola Pengguna

Actor:

- Admin Desa

## UC-005 — Kelola Lokasi

Actor:

- Admin Desa

## UC-009 — Catat Zakat Fitrah

### Actor

Petugas Lokasi

### Goal

Mencatat pembayaran zakat fitrah yang dilakukan oleh
warga/keluarga.

### Preconditions

- Petugas telah login.
- Akun petugas aktif.
- Petugas telah memiliki lokasi.
- Periode pendataan sedang aktif.

### Main Flow

1. Petugas membuka menu pendataan zakat.
2. Petugas memilih Tambah Data.
3. Sistem menampilkan formulir.
4. Petugas memasukkan nama kepala keluarga.
5. Petugas memasukkan jumlah jiwa.
6. Petugas memasukkan jumlah beras dan/atau uang.
7. Petugas mengonfirmasi tanggal pembayaran.
8. Petugas memilih Simpan.
9. Sistem melakukan validasi.
10. Sistem menyimpan data.
11. Sistem menampilkan informasi bahwa data berhasil disimpan.

### Alternative Flow

#### Jumlah jiwa tidak valid

Jika jumlah jiwa kurang dari satu, sistem menolak data
dan menampilkan pesan kesalahan.

#### Pembayaran kosong

Jika beras dan uang tidak diisi, sistem menolak data.

### Postconditions

Data pembayaran zakat tersimpan pada lokasi milik petugas.
