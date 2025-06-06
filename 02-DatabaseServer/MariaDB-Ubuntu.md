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
