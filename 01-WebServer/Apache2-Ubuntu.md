# Apache2
> Apache2 (nama lengkapnya Apache HTTP Server) adalah web server open-source yang paling banyak dipake di dunia buat ngehost website, baik yang statis (HTML biasa) maupun dinamis (kayak PHP, WordPress, Laravel, dll).

> Dirilis sama Apache Software Foundation, Apache ini udah dipake sejak tahun 1995, dan sampai sekarang masih jadi tulang punggung banyak server di internet — stabil, fleksibel, dan bisa di-custom abis.

## Modul populer di apache2

| Modul          | Fungsi                                  |
| -------------- | --------------------------------------- |
| `mod_ssl`      | Aktifin HTTPS pakai sertifikat SSL      |
| `mod_rewrite`  | Bikin URL cantik, redirect, dll         |
| `mod_headers`  | Tambah header keamanan kayak HSTS, CSP  |
| `mod_security` | WAF (Web Application Firewall)          |
| `mod_php`      | Jalankan skrip PHP langsung dari Apache |
| `mod_proxy`    | Jadikan Apache sebagai reverse proxy    |

## Installasi Apache2
```bash
sudo apt update
sudo apt install apache2 -y
```

> *Instal paket utama Apache2.*

### Cek & Aktifkan Service

```bash
sudo systemctl status apache2
sudo systemctl enable --now apache2
```

> *Pastikan service jalan otomatis tiap booting.*

### Aktifkan Firewall UFW

```bash
sudo ufw allow 'Apache Full'
sudo ufw enable
```

> *Allow port 80 & 443 buat HTTP & HTTPS.*
### Buka Browser

> Akses `http://ip-server`

> Kalau muncul **"Apache2 Ubuntu Default Page"**, artinya berhasil!
### Aktifkan Modul Pendukung (SSL, Rewrite)

```bash
sudo a2enmod rewrite ssl headers
sudo systemctl restart apache2
```

> *Module rewrite penting buat .htaccess dan SSL redirect.*

### Install PHP Support

```bash
sudo apt install php libapache2-mod-php php-mysql -y
```

Test file:

```bash
sudo nano /var/www/html/info.php
```

Isi:

```php
<?php phpinfo(); ?>
```

Lalu akses: `http://ip-server/info.php`

## UserDir
> Ini fitur yang bikin tiap user di sistem bisa punya direktory web sendiri.

### Aktifkan modul userdir
```bash
a2enmod userdir

systemctl restart apache2
```
### Buat Direktory Web user
```
adduser dapit
su - dapit
chmod 711 $HOME
mkdir public_html
chmod 755 public_html
cat << EOF > public_html/index.html
<h1>Welcome to $(whoami)'s site.</h1>
EOF
```
### Reload service
```bash
systemctl reload apache2
```
### Testing
```bash
http://ip-address/~username

http://192.168.50.1/~dapit
```

## Basic Auth
> Ini cara buat ngunci akses ke folder atau website pake username dan password. Biasanya dipakai buat folder admin, panel sensitif, testing server.

### Install `apache2-utils`
```bash
apt install apache2-utils -y
```

### Buat file password
```bash
sudo htpasswd -c /etc/apache2/.htpasswd dapit
```
> File ini berisi username & hash password

### Edit `.htaccess`
```bash
cat << EOF > public_html/.htaccess
AuthType Basic
AuthName "Basic Authentication"
AuthUserFile /home/dapit/public_html/.htpasswd
Require valid-user
EOF
```
> ini akan membuat access untuk web user

`Jangan lupa aktifkan .htaccess di vhost:`
```bash
AllowOverride All
```

### Restart Apache
`systemctl reload apache2`

## Konfigurasi VirtualHost
Buat File VirtualHost:

```bash
sudo nano /etc/apache2/sites-available/0xbyalak.com.conf
```

Isi:

```apacheconf
<VirtualHost *:80>
    ServerAdmin admin@0xbyalak.com
    ServerName 0xbyalak.com
    DocumentRoot /var/www/project

    <Directory /var/www/project>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/project-error.log
    CustomLog ${APACHE_LOG_DIR}/project-access.log combined
</VirtualHost>
```
### Aktifkan VirtualHost

```bash
sudo a2ensite oxbyalak.com.conf
sudo systemctl reload apache2
```

### Aktifkan HTTPS

**(Gunakan Let's Encrypt)**

```bash
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache
```
Ikuti wizardnya, nanti otomatis config HTTPS + redirect 80 → 443.

## HARDENING

### Nonaktifkan Info Versi

```bash
sudo nano /etc/apache2/conf-available/security.conf
```

Ubah:

```
ServerTokens Prod
ServerSignature Off
```

> *Sembunyikan info versi Apache agar tidak bisa fingerprinting.*

---

### Aktifkan HTTP Security Header

Edit vhost:

```apacheconf
<IfModule mod_headers.c>
  Header always set X-Frame-Options "SAMEORIGIN"
  Header always set X-XSS-Protection "1; mode=block"
  Header always set X-Content-Type-Options "nosniff"
  Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
</IfModule>
```

> *Cegah serangan clickjacking, XSS, sniffing, downgrade TLS.*

---

### Install dan Konfigurasi ModSecurity (WAF)

```bash
sudo apt install libapache2-mod-security2 -y
sudo a2enmod security2
sudo systemctl restart apache2
```

Aktifkan ruleset OWASP:

```bash
sudo cp /usr/share/modsecurity-crs /etc/modsecurity-crs -r
sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
sudo nano /etc/modsecurity/modsecurity.conf
```

Ubah:

```
SecRuleEngine On
```

> *ModSecurity = firewall aplikasi web (WAF) yang powerful.*

---

### Set Permission dan Ownership

```bash
sudo chown -R www-data:www-data /var/www/project.local
sudo chmod -R 755 /var/www/project.local
```

> *Biar webserver punya akses tapi tetap aman dari eksekusi aneh-aneh.*
