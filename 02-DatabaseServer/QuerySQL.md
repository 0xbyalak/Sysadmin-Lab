# SQL Cheatsheet Lengkap

>Panduan ringkas ini dibuat buat kalian yang pengen cepat paham dan langsung praktik query SQL — mulai dari CRUD, join, sampai manajemen user. Setiap perintah dikasih penjelasan singkat biar nggak cuma hafal, tapi juga ngerti.


## 1. CRUD Operations

### SELECT - Ambil data dari tabel
```sql
SELECT * FROM users;  -- Ambil semua kolom dari tabel users
SELECT name, email FROM users WHERE status = 'active';  -- Filter hanya user yang aktif
SELECT COUNT(*) FROM orders WHERE status = 'shipped';  -- Hitung jumlah order yang dikirim
SELECT DISTINCT country FROM customers;  -- Ambil negara unik dari customers
```

### INSERT - Tambah data baru
```sql
INSERT INTO users (name, email, status) VALUES ('Davit', 'davit@mail.com', 'active');
```
```sql
-- Tambah banyak data sekaligus
INSERT INTO users (name, email) VALUES 
('User1', 'u1@mail.com'),
('User2', 'u2@mail.com');
```

### UPDATE - Ubah data yang ada
```sql
UPDATE users SET status = 'inactive' WHERE last_login < '2024-01-01';  -- Inaktivasi user lama
```
```sql
UPDATE products SET stock = 0, updated_at = NOW() WHERE discontinued = 1;  -- Set stok nol kalau sudah discontinued
```

### DELETE - Hapus data
```sql
DELETE FROM users WHERE status = 'inactive';  -- Hapus user nonaktif
```
```sql
DELETE FROM users;  -- Hapus semua data (hati-hati)
TRUNCATE TABLE users;  -- Hapus semua data dan reset auto_increment
```

---

## 2. Manajemen Tabel

### CREATE TABLE - Buat tabel baru
```sql
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100) UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### ALTER TABLE - Ubah struktur tabel
```sql
ALTER TABLE users ADD COLUMN phone VARCHAR(15);  -- Tambah kolom
ALTER TABLE users MODIFY COLUMN email VARCHAR(150);  -- Ubah panjang kolom
ALTER TABLE users DROP COLUMN phone;  -- Hapus kolom
```

### DROP TABLE - Hapus tabel dari database
```sql
DROP TABLE users;  -- Hapus seluruh tabel beserta datanya
```

---

## 3. JOIN

### INNER JOIN - Ambil data yang cocok di kedua tabel
```sql
SELECT orders.id, customers.name
FROM orders
INNER JOIN customers ON orders.customer_id = customers.id;
```

### LEFT JOIN - Ambil semua data dari kiri, cocokkan dengan kanan
```sql
SELECT users.name, orders.id
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

### RIGHT JOIN - Kebalikan dari LEFT JOIN
```sql
SELECT users.name, orders.id
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;
```

### FULL OUTER JOIN - Gabungkan semua data (butuh UNION)
```sql
SELECT * FROM table1
LEFT JOIN table2 ON table1.id = table2.id
UNION
SELECT * FROM table1
RIGHT JOIN table2 ON table1.id = table2.id;
```

---

## 4. Filtering & Sorting
```sql
SELECT * FROM products WHERE price > 1000 AND stock > 0;  -- Filter pakai kondisi
SELECT * FROM products ORDER BY price DESC;  -- Urutkan dari harga terbesar
SELECT * FROM users LIMIT 10 OFFSET 20;  -- Ambil 10 data, mulai dari data ke-21
```

---

## 5. Aggregate Functions
```sql
SELECT COUNT(*) FROM users;  -- Hitung semua baris
SELECT AVG(price) FROM products;  -- Rata-rata harga
SELECT SUM(quantity) FROM orders;  -- Total quantity
SELECT MIN(price), MAX(price) FROM products;  -- Harga minimum dan maksimum
```

### GROUP BY - Kelompokkan hasil
```sql
SELECT country, COUNT(*) FROM customers GROUP BY country;  -- Hitung pelanggan per negara
```

### HAVING - Filter hasil GROUP BY
```sql
SELECT user_id, COUNT(*) as total
FROM orders
GROUP BY user_id
HAVING total > 5;  -- Hanya tampilkan user yang punya lebih dari 5 order
```

---

## 6. Subquery

### Di SELECT
```sql
SELECT name, (SELECT COUNT(*) FROM orders WHERE user_id = users.id) as order_count
FROM users;  -- Tambahkan kolom jumlah order per user
```

### Di WHERE
```sql
SELECT * FROM products
WHERE category_id IN (SELECT id FROM categories WHERE name = 'Elektronik');
```

---

## 7. Manajemen User MariaDB / MySQL
```sql
CREATE USER 'dapit'@'localhost' IDENTIFIED BY 'PasswordKuat123!';  -- Buat user baru
GRANT ALL PRIVILEGES ON db_name.* TO 'dapit'@'localhost';  -- Kasih akses
ALTER USER 'dapit'@'localhost' IDENTIFIED BY 'BaruBanget123!';  -- Ganti password
REVOKE ALL PRIVILEGES ON db_name.* FROM 'dapit'@'localhost';  -- Cabut akses
DROP USER 'dapit'@'localhost';  -- Hapus user
```

---

## 8. Tips Keamanan & Optimasi

- Hindari `SELECT *` di production — pilih kolom yang dibutuhin aja
- Selalu pakai `WHERE` saat `UPDATE`/`DELETE`
- Gunakan `PREPARED STATEMENTS` untuk hindari SQL Injection
- Aktifkan `slow_query_log` untuk debug query lambat
- Gunakan index untuk kolom yang sering dipakai di `WHERE`, `JOIN`, `ORDER BY`
- Hapus user bawaan yang tidak terpakai di MariaDB (`anonymous`, `test`)

---

## 9. Miscellaneous
```sql
DESCRIBE users;  -- Lihat struktur tabel
SHOW COLUMNS FROM users;  -- Sama seperti DESCRIBE
SHOW DATABASES;  -- Lihat semua database
USE nama_database;  -- Pindah ke database tertentu
SHOW TABLES;  -- Lihat semua tabel di database aktif
```

### Backup & Restore via CLI
```bash
mysqldump -u root -p nama_database > backup.sql  # Backup
mysql -u root -p nama_database < backup.sql      # Restore
```

---

## 10. View, Indexing, dan Performance

### VIEW - Buat virtual table dari query
```sql
CREATE VIEW active_users AS
SELECT id, name, email FROM users WHERE status = 'active';

SELECT * FROM active_users;  -- Pakai view seperti tabel biasa
```

### INDEX - Percepat pencarian dan filter data
```sql
CREATE INDEX idx_email ON users(email);  -- Buat index di kolom email
DROP INDEX idx_email ON users;  -- Hapus index
```

### CEK PERFORMANCE QUERY
```sql
EXPLAIN SELECT * FROM users WHERE email = 'davit@mail.com';  -- Lihat cara MySQL eksekusi query
SHOW STATUS LIKE 'Handler_read%';  -- Statistik baca handler
```
