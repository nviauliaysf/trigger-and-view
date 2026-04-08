🚀 ServiceTrack DBMS
Sistem Manajemen Layanan & Inventaris Berbasis MySQL

📌 Deskripsi Project

ServiceTrack DBMS merupakan sebuah sistem database berbasis MySQL yang dirancang untuk mengelola proses layanan pelanggan serta penggunaan inventaris secara terintegrasi. Sistem ini dibuat untuk mensimulasikan bagaimana sebuah layanan servis menangani transaksi pelanggan sekaligus mengontrol penggunaan barang secara otomatis.

Dalam implementasinya, setiap pelanggan yang melakukan layanan akan tercatat dalam tabel servis, kemudian setiap barang yang digunakan selama proses layanan akan disimpan pada tabel detail_servis. Data barang tersebut diambil dari tabel sparepart yang juga menyimpan informasi stok dan harga. Semua tabel ini saling terhubung menggunakan relasi sehingga membentuk satu kesatuan sistem yang utuh.

Keunggulan utama dari project ini adalah penggunaan trigger yang memungkinkan database bekerja secara otomatis. Database tidak hanya menyimpan data, tetapi juga melakukan validasi, perhitungan, dan pengelolaan data secara mandiri. Selain itu, digunakan juga view untuk mempermudah pengambilan data laporan tanpa harus menulis query yang kompleks.

Dengan pendekatan ini, database berperan sebagai pusat logika sistem, bukan hanya sebagai tempat penyimpanan data.

⚡ Fitur Utama

Sistem ini dibangun dengan struktur relasional yang memastikan setiap data memiliki keterkaitan yang jelas. Ketika sebuah data pelanggan dibuat, data tersebut dapat digunakan dalam transaksi servis, dan setiap transaksi akan memiliki detail penggunaan barang yang terhubung langsung dengan data sparepart. Relasi ini membantu menjaga konsistensi data dan mempermudah proses pencarian informasi.

Selain itu, sistem ini memanfaatkan trigger untuk mengatur berbagai proses otomatis. Saat data dimasukkan ke dalam tabel detail_servis, sistem secara otomatis akan mengecek apakah stok mencukupi. Jika stok tidak mencukupi, maka data tidak akan disimpan. Namun jika stok tersedia, sistem langsung mengurangi stok dan menambahkan total biaya ke dalam tabel servis tanpa perlu perhitungan manual.

Penggunaan view dalam sistem ini juga memberikan kemudahan dalam menampilkan data. View berfungsi sebagai tabel virtual yang menyimpan hasil query tertentu, sehingga pengguna dapat melihat laporan tanpa harus menulis query join yang panjang setiap kali dibutuhkan.

Dengan adanya kombinasi antara relasi, trigger, dan view, sistem ini menjadi lebih efisien, aman, dan siap digunakan dalam skenario nyata.


🗄️ Langkah-Langkah Setup Database

Langkah pertama yang harus dilakukan adalah membuat database sebagai tempat penyimpanan seluruh data. Setelah database dibuat, database tersebut perlu diaktifkan agar semua perintah SQL yang dijalankan akan tersimpan di dalamnya.

sql
CREATE DATABASE bengkel;
USE bengkel;

Perintah ini akan membuat database bernama *bengkel* dan langsung menggunakannya.

🧱 Struktur Tabel dan Penjelasannya

Tahap berikutnya adalah membuat tabel-tabel yang menjadi fondasi sistem. Tabel pertama yang dibuat adalah tabel pelanggan yang berfungsi untuk menyimpan data pelanggan.

sql
CREATE TABLE pelanggan (
  id_pelanggan INT AUTO_INCREMENT PRIMARY KEY,
  nama VARCHAR(100));

Tabel ini memiliki kolom id_pelanggan sebagai primary key yang akan terisi otomatis, serta kolom nama untuk menyimpan nama pelanggan.

Selanjutnya dibuat tabel servis yang digunakan untuk mencatat transaksi layanan. Tabel ini memiliki hubungan dengan tabel pelanggan melalui id_pelanggan.

sql
CREATE TABLE servis (
  id_servis INT AUTO_INCREMENT PRIMARY KEY,
  id_pelanggan INT,
  total_biaya INT DEFAULT 0,
  FOREIGN KEY (id_pelanggan) REFERENCES pelanggan(id_pelanggan));

Kolom total_biaya digunakan untuk menyimpan total biaya layanan yang akan diperbarui secara otomatis oleh trigger.

Tabel berikutnya adalah tabel sparepart yang berfungsi untuk menyimpan data barang beserta stok dan harganya.

sql
CREATE TABLE sparepart (
  id_sparepart INT AUTO_INCREMENT PRIMARY KEY,
  nama_sparepart VARCHAR(100),
  stok INT,
  harga INT);

Tabel ini sangat penting karena menjadi sumber data barang yang digunakan dalam transaksi servis.

Terakhir adalah tabel detail_servis yang digunakan untuk mencatat barang apa saja yang digunakan dalam setiap transaksi.

sql
CREATE TABLE detail_servis (
  id_detail INT AUTO_INCREMENT PRIMARY KEY,
  id_servis INT,
  id_sparepart INT,
  jumlah INT,
  harga INT,
  FOREIGN KEY (id_servis) REFERENCES servis(id_servis),
  FOREIGN KEY (id_sparepart) REFERENCES sparepart(id_sparepart));

Tabel ini menghubungkan servis dengan sparepart dan menjadi pusat perhitungan dalam sistem.

🧪 Data Awal untuk Pengujian

Untuk memastikan sistem dapat langsung digunakan, data awal dimasukkan ke dalam tabel.

sql
INSERT INTO pelanggan (nama) VALUES ('Budi'), ('Andi');

INSERT INTO sparepart (nama_sparepart, stok, harga) VALUES
('Oli', 10, 10000),
('Busi', 5, 5000);

INSERT INTO servis (id_pelanggan) VALUES
(1),
(2);

Data ini digunakan untuk melakukan pengujian trigger dan view.

⚡ Trigger dan Penjelasannya

Trigger merupakan bagian inti dari sistem ini karena berfungsi untuk menjalankan logika secara otomatis.

Trigger pertama digunakan untuk memvalidasi stok sebelum data dimasukkan. Sistem akan mengecek apakah jumlah barang yang diminta melebihi stok yang tersedia. Jika iya, maka transaksi akan dibatalkan.

sql 
DELIMITER //
CREATE TRIGGER before_insert_cek_stok
BEFORE INSERT ON detail_servis
FOR EACH ROW
BEGIN
  DECLARE sisa INT;

  SELECT stok INTO sisa 
  FROM sparepart
  WHERE id_sparepart = NEW.id_sparepart;

  IF sisa < NEW.jumlah THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Stok tidak cukup';
  END IF;
END;
//
DELIMITER ;

Setelah data berhasil dimasukkan, trigger berikutnya akan langsung berjalan untuk mengurangi stok dan menambahkan total biaya ke dalam tabel servis.

sql
DELIMITER //
CREATE TRIGGER after_insert_update
AFTER INSERT ON detail_servis
FOR EACH ROW
BEGIN
  UPDATE sparepart 
  SET stok = stok - NEW.jumlah
  WHERE id_sparepart = NEW.id_sparepart;

  UPDATE servis
  SET total_biaya = total_biaya + (NEW.jumlah * NEW.harga)
  WHERE id_servis = NEW.id_servis;
END;
//
DELIMITER ;

Ketika data diubah, sistem kembali melakukan validasi agar perubahan tersebut tidak menyebabkan stok menjadi negatif.

sql
DELIMITER //
CREATE TRIGGER before_update_cek
BEFORE UPDATE ON detail_servis
FOR EACH ROW
BEGIN
  DECLARE sisa INT;

  SELECT stok INTO sisa 
  FROM sparepart
  WHERE id_sparepart = NEW.id_sparepart;

  IF sisa < (NEW.jumlah - OLD.jumlah) THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Stok tidak cukup untuk update';
  END IF;
END;
//
DELIMITER ;

Jika update berhasil, maka trigger berikutnya akan menyesuaikan stok dan total biaya agar tetap sesuai dengan data terbaru.

sql
DELIMITER //
CREATE TRIGGER after_update_fix
AFTER UPDATE ON detail_servis
FOR EACH ROW
BEGIN
  UPDATE sparepart
  SET stok = stok + OLD.jumlah - NEW.jumlah
  WHERE id_sparepart = NEW.id_sparepart;

  UPDATE servis
  SET total_biaya = total_biaya +
  ((NEW.jumlah * NEW.harga) - (OLD.jumlah * OLD.harga))
  WHERE id_servis = NEW.id_servis;
END;
//
DELIMITER ;

Sebelum data dihapus, sistem akan memastikan bahwa data tersebut valid untuk dihapus.

sql 
DELIMITER //
CREATE TRIGGER before_delete_cek
BEFORE DELETE ON detail_servis
FOR EACH ROW
BEGIN
  IF OLD.jumlah <= 0 THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Data tidak valid untuk dihapus';
  END IF;
END;
//
DELIMITER ;

Setelah data dihapus, trigger terakhir akan mengembalikan stok dan mengurangi total biaya servis.

sql 
DELIMITER //
CREATE TRIGGER after_delete_fix
AFTER DELETE ON detail_servis
FOR EACH ROW
BEGIN
  UPDATE sparepart 
  SET stok = stok + OLD.jumlah
  WHERE id_sparepart = OLD.id_sparepart;

  UPDATE servis
  SET total_biaya = total_biaya - (OLD.jumlah * OLD.harga)
  WHERE id_servis = OLD.id_servis;
END;
//
DELIMITER ;

Dengan adanya trigger ini, semua proses berjalan secara otomatis dan konsisten.

👁️ View dan Penjelasannya

View digunakan untuk menyederhanakan pengambilan data.

View pertama digunakan untuk menampilkan laporan total biaya servis beserta nama pelanggan.

sql 
CREATE VIEW view_total_servis AS
SELECT 
  s.id_servis,
  p.nama AS nama_pelanggan,
  s.total_biaya
FROM servis s
JOIN pelanggan p ON s.id_pelanggan = p.id_pelanggan;

View kedua digunakan untuk melihat kondisi stok barang secara langsung.

sql 
CREATE VIEW view_stok_sparepart AS
SELECT 
  id_sparepart,
  nama_sparepart,
  stok,
  harga
FROM sparepart;

Dengan adanya view ini, pengguna tidak perlu menulis query yang kompleks.

🧪 Contoh Penggunaan

Contoh berikut menunjukkan bagaimana sistem bekerja secara otomatis saat data dimasukkan.

sql 
INSERT INTO detail_servis (id_servis, id_sparepart, jumlah, harga)
VALUES (1, 1, 2, 10000);

Saat perintah ini dijalankan, sistem akan langsung mengecek stok, mengurangi stok, dan menambahkan total biaya.

Jika jumlah yang dimasukkan melebihi stok, maka sistem akan menghasilkan error.

sql
INSERT INTO detail_servis (id_servis, id_sparepart, jumlah, harga)
VALUES (1, 1, 999, 10000);

Output:
Stok tidak cukup

🎯 Kesimpulan

Project ini menunjukkan bahwa database dapat digunakan sebagai pusat logika sistem, bukan hanya sebagai tempat penyimpanan data. Dengan memanfaatkan trigger dan view, sistem menjadi lebih otomatis, aman, dan efisien.

Dibandingkan database sederhana, sistem ini memiliki kemampuan untuk melakukan validasi, perhitungan, serta pengelolaan data secara mandiri. Hal ini menjadikannya lebih siap untuk digunakan dalam aplikasi nyata.

