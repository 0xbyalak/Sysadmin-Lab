# MariaDB
> Sistem manajemen basis data relasional (RDBMS) yang dikembangkan sebagai fork dari MySQL, dan dirancang supaya tetap gratis, open-source, dan community-driven.

## Installasi
```bash
apt update
apt install mariadb-server -y
```
Cek service & aktifkan:
```bash
systemctl status mariadb
systemctl enable --now mariadb
```
> Pastikan Service Active (running).

## Basic secure installation
```bash
mysql_secure_installation
```
`Jawaban:`

- Set root password → Yes
- Remove anonymous users → Yes
- Disallow root login remotely → Yes
- Remove test database → Yes
- Reload privilege tables → Yes

## Ganti Unix Socket Auth
```bash
sudo mysql -u root
```
> Kalau bisa masuk tanpa password, berarti pakai unix_socket.
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PasswordBaru123!';
FLUSH PRIVILEGES;
```
`exit`
## Hardening konfigurasi
Edit:
```bash
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Tambahkan di bawah [mysqld]:
```bash
bind-address = 192.168.1.10         # Ganti ke IP internal atau public yang boleh akses
skip-symbolic-links                 # Cegah eksploitasi file
skip-name-resolve                   # Pakai IP aja, lebih cepat & aman
local-infile=0                      # Cegah LOAD DATA LOCAL dari file (sering dipakai buat serangan)
max_connections=100
connect_timeout=5
wait_timeout=600
interactive_timeout=600
sql_mode=STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION    # Cegah LOAD DATA LOCAL dari file (sering dipakai buat serangan
```
`save & keluar`

## Konfigurasi firewall
Misal hanya izinkan akses dari subnet tertentu:
```bash
ufw allow from 192.168.1.0/24 to any port 3306
ufw enable
```

## Restart service
```bash
systemctl restart mariadb
systemctl status mariadb
```
