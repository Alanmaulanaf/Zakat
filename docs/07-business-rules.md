| ID         | Business Rule                                                                                                                         |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **BR-001** | Jumlah jiwa dalam satu pencatatan zakat harus minimal 1.                                                                              |
| **BR-002** | Setiap pencatatan zakat harus memiliki pembayaran berupa beras, uang, atau keduanya sesuai proses yang berlaku.                       |
| **BR-003** | Jumlah beras tidak boleh bernilai negatif.                                                                                            |
| **BR-004** | Nominal uang tidak boleh bernilai negatif.                                                                                            |
| **BR-005** | Petugas hanya dapat mengelola data pada lokasi yang ditugaskan kepadanya.                                                             |
| **BR-006** | Setiap pencatatan zakat harus terkait dengan satu lokasi pembayaran.                                                                  |
| **BR-007** | Setiap pencatatan zakat harus terkait dengan satu periode zakat fitrah.                                                               |
| **BR-008** | Pendataan hanya dapat dilakukan pada periode yang sedang aktif, kecuali pengguna yang memiliki kewenangan khusus.                     |
| **BR-009** | Besaran zakat per jiwa tidak boleh di-hard-code dan harus dapat dikonfigurasi berdasarkan periode.                                    |
| **BR-010** | Perubahan besaran zakat pada satu periode tidak boleh mengubah data historis periode sebelumnya.                                      |
| **BR-011** | Jumlah KK pada rekap dihitung berdasarkan jumlah pencatatan keluarga yang valid, bukan jumlah jiwa.                                   |
| **BR-012** | Total jiwa merupakan penjumlahan jumlah jiwa dari seluruh pencatatan yang memenuhi kriteria rekap.                                    |
| **BR-013** | Total beras merupakan penjumlahan jumlah beras dari seluruh pencatatan yang memenuhi kriteria rekap.                                  |
| **BR-014** | Total uang merupakan penjumlahan nominal uang dari seluruh pencatatan yang memenuhi kriteria rekap.                                   |
| **BR-015** | Data yang sudah dikirim (`SUBMITTED`) tidak dapat diubah secara bebas oleh Petugas Lokasi.                                            |
| **BR-016** | Data berstatus `REVISION_REQUIRED` dapat diperbaiki oleh Petugas Lokasi sebelum dikirim kembali.                                      |
| **BR-017** | Data yang telah `APPROVED` tidak dapat diubah oleh Petugas Lokasi.                                                                    |
| **BR-018** | Hanya pengguna yang memiliki kewenangan yang dapat menyetujui atau meminta perbaikan rekap.                                           |
| **BR-019** | Permintaan perbaikan harus disertai alasan/keterangan.                                                                                |
| **BR-020** | Rekap resmi desa hanya menggunakan data yang telah memenuhi status final yang ditentukan, misalnya `APPROVED`.                        |
| **BR-021** | Setiap perubahan penting terhadap data zakat harus dapat dilacak melalui audit log.                                                   |
| **BR-022** | Audit log tidak boleh dapat diedit oleh Petugas Lokasi.                                                                               |
| **BR-023** | Data periode sebelumnya harus tetap dapat dilihat sebagai histori meskipun periode baru telah dibuat.                                 |
| **BR-024** | Lokasi yang dinonaktifkan tidak dapat digunakan untuk pencatatan baru, tetapi data historisnya tetap disimpan.                        |
| **BR-025** | Akun pengguna yang dinonaktifkan tidak dapat melakukan login, tetapi data historis yang dibuat pengguna tersebut tetap dipertahankan. |
| **BR-026** | Penghapusan data transaksi tidak boleh dilakukan secara permanen tanpa mekanisme yang menjaga histori perubahan.                      |
