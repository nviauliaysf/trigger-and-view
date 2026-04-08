# 🚀 ServiceTrack DBMS
### Sistem Manajemen Layanan & Inventaris Berbasis MySQL

---

## 📌 Deskripsi Project

**ServiceTrack DBMS** adalah sebuah sistem database berbasis MySQL yang dirancang untuk mengelola proses layanan pelanggan serta penggunaan inventaris secara terstruktur dan otomatis. Sistem ini menggambarkan bagaimana sebuah layanan dapat mencatat transaksi, mengelola penggunaan barang, serta menjaga konsistensi data tanpa harus bergantung sepenuhnya pada aplikasi.

Dalam sistem ini, data pelanggan disimpan pada tabel pelanggan, kemudian setiap aktivitas layanan dicatat pada tabel servis. Setiap layanan dapat memiliki beberapa penggunaan barang yang dicatat dalam tabel detail_servis. Barang yang digunakan berasal dari tabel sparepart yang menyimpan informasi stok dan harga.

Yang membuat sistem ini lebih unggul dibanding database biasa adalah penggunaan trigger. Trigger memungkinkan database untuk menjalankan logika secara otomatis, seperti validasi stok, pengurangan stok, serta perhitungan total biaya. Selain itu, view digunakan untuk menyederhanakan proses pengambilan data laporan sehingga lebih mudah dibaca dan digunakan.

Dengan pendekatan ini, database tidak hanya berfungsi sebagai tempat penyimpanan, tetapi juga sebagai pengatur logika sistem.

---

## ⚡ Fitur Utama

Sistem ini menggunakan relasi antar tabel yang memastikan setiap data memiliki hubungan yang jelas. Data pelanggan terhubung dengan data servis, kemudian data servis terhubung dengan detail penggunaan barang. Struktur ini membuat data menjadi lebih rapi, konsisten, dan mudah ditelusuri.

Selain itu, sistem memanfaatkan trigger untuk menangani proses otomatis. Setiap kali data dimasukkan ke dalam detail_servis, sistem akan langsung mengecek apakah stok mencukupi. Jika tidak, maka proses akan dibatalkan. Jika stok cukup, sistem akan langsung mengurangi stok dan menambahkan total biaya tanpa perlu perhitungan manual.

View juga digunakan untuk mempermudah pengambilan data. Dengan view, pengguna tidak perlu menulis query join yang panjang karena data sudah disiapkan dalam bentuk yang lebih sederhana.

---

## 🗄️ Langkah-Langkah Setup Database

Langkah pertama adalah membuat database dan mengaktifkannya:

```sql
CREATE DATABASE bengkel;
USE bengkel;
````

Perintah ini akan membuat database baru dan menjadikannya aktif.

---

## 🧱 Struktur Tabel dan Penjelasannya

Tabel pelanggan digunakan untuk menyimpan data pelanggan yang melakukan layanan.

```sql
CREATE TABLE pelanggan (
  id_pelanggan INT AUTO_INCREMENT PRIMARY KEY,
  nama VARCHAR(100)
);
```

Tabel servis digunakan untuk mencatat transaksi layanan yang dilakukan oleh pelanggan.

```sql
CREATE TABLE servis (
  id_servis INT AUTO_INCREMENT PRIMARY KEY,
  id_pelanggan INT,
  total_biaya INT DEFAULT 0,
  FOREIGN KEY (id_pelanggan) REFERENCES pelanggan(id_pelanggan)
);
```

Tabel sparepart menyimpan data barang yang tersedia, termasuk stok dan harga.

```sql
CREATE TABLE sparepart (
  id_sparepart INT AUTO_INCREMENT PRIMARY KEY,
  nama_sparepart VARCHAR(100),
  stok INT,
  harga INT
);
```

Tabel detail_servis berfungsi untuk mencatat barang yang digunakan dalam setiap transaksi servis.

```sql
CREATE TABLE detail_servis (
  id_detail INT AUTO_INCREMENT PRIMARY KEY,
  id_servis INT,
  id_sparepart INT,
  jumlah INT,
  harga INT,
  FOREIGN KEY (id_servis) REFERENCES servis(id_servis),
  FOREIGN KEY (id_sparepart) REFERENCES sparepart(id_sparepart)
);
```

---

## 🧪 Data Awal

Data awal digunakan untuk pengujian sistem agar bisa langsung dicoba.

```sql
INSERT INTO pelanggan (nama) VALUES ('Budi'), ('Andi');

INSERT INTO sparepart (nama_sparepart, stok, harga) VALUES
('Oli', 10, 10000),
('Busi', 5, 5000);

INSERT INTO servis (id_pelanggan) VALUES
(1),
(2);
```

---

## ⚡ Trigger dan Penjelasannya

Trigger pertama digunakan untuk mengecek apakah stok mencukupi sebelum data dimasukkan.

```sql
DELIMITER //
CREATE TRIGGER before_insert_cek_stok
BEFORE INSERT ON detail_servis
FOR EACH ROW
BEGIN
  DECLARE sisa INT;
  SELECT stok INTO sisa FROM sparepart WHERE id_sparepart = NEW.id_sparepart;

  IF sisa < NEW.jumlah THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Stok tidak cukup';
  END IF;
END;
//
DELIMITER ;
```

Trigger berikutnya berjalan setelah data dimasukkan untuk mengurangi stok dan menambahkan total biaya.

```sql
DELIMITER //
CREATE TRIGGER after_insert_update
AFTER INSERT ON detail_servis
FOR EACH ROW
BEGIN
  UPDATE sparepart SET stok = stok - NEW.jumlah WHERE id_sparepart = NEW.id_sparepart;

  UPDATE servis
  SET total_biaya = total_biaya + (NEW.jumlah * NEW.harga)
  WHERE id_servis = NEW.id_servis;
END;
//
DELIMITER ;
```

Trigger update memastikan perubahan data tidak menyebabkan stok menjadi negatif.

```sql
DELIMITER //
CREATE TRIGGER before_update_cek
BEFORE UPDATE ON detail_servis
FOR EACH ROW
BEGIN
  DECLARE sisa INT;
  SELECT stok INTO sisa FROM sparepart WHERE id_sparepart = NEW.id_sparepart;

  IF sisa < (NEW.jumlah - OLD.jumlah) THEN
    SIGNAL SQLSTATE '45000'
    SET MESSAGE_TEXT = 'Stok tidak cukup untuk update';
  END IF;
END;
//
DELIMITER ;
```

Setelah update, sistem akan menyesuaikan stok dan total biaya.

```sql
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
```

Trigger delete memastikan data valid sebelum dihapus.

```sql
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
```

Setelah data dihapus, sistem akan mengembalikan stok dan menyesuaikan total biaya.

```sql
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
```

---

## 👁️ View dan Penjelasannya

View pertama digunakan untuk menampilkan laporan total biaya servis berdasarkan pelanggan.

```sql
CREATE VIEW view_total_servis AS
SELECT 
  s.id_servis,
  p.nama AS nama_pelanggan,
  s.total_biaya
FROM servis s
JOIN pelanggan p ON s.id_pelanggan = p.id_pelanggan;
```

View kedua digunakan untuk melihat kondisi stok barang.

```sql
CREATE VIEW view_stok_sparepart AS
SELECT 
  id_sparepart,
  nama_sparepart,
  stok,
  harga
FROM sparepart;
```

---

## 🧪 Contoh Penggunaan

```sql
INSERT INTO detail_servis (id_servis, id_sparepart, jumlah, harga)
VALUES (1, 1, 2, 10000);
```

Jika jumlah melebihi stok:

```sql
INSERT INTO detail_servis (id_servis, id_sparepart, jumlah, harga)
VALUES (1, 1, 999, 10000);
```

Output:

```
Stok tidak cukup
```

---

## 🎯 Kesimpulan

Sistem ini menunjukkan bahwa database dapat digunakan sebagai pusat logika, bukan hanya sebagai tempat penyimpanan data. Dengan adanya trigger dan view, sistem mampu berjalan secara otomatis, menjaga konsistensi data, serta mengurangi kesalahan manusia.

---

## 🚀 Penutup

Project ini sangat cocok dijadikan portofolio karena telah mengimplementasikan konsep database tingkat lanjut seperti relasi, trigger, dan view. Sistem ini juga sudah mendekati implementasi nyata dalam dunia industri.

---
