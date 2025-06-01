###  **Step-by-step Setup Cloudflare Tunnel:**
Cloudflare Tunnel adalah layanan dari Cloudflare yang memungkinkan kita mengakses server lokal secara aman dari internet tanpa perlu buka port di router atau punya IP publik. Dengan menjalankan agen cloudflared di server, koneksi terenkripsi akan dibuat ke jaringan Cloudflare, lalu pengguna bisa mengakses layanan lokal lewat domain yang diatur di DNS Cloudflare. Cocok banget buat deploy web app, dashboard, atau service internal secara cepat dan aman.

> *Pastikan domain kamu aktif di Cloudflare.*

#### 1. **Install `cloudflared`**

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

#### 2. **Login ke Cloudflare**

```bash
cloudflared login
```

Ini akan membuka browser untuk otorisasi domain kamu jika copy url yang ada dan paste ke browser. Pilih domain `klandestin.site`.

#### 3. **Buat tunnel**

```bash
cloudflared tunnel create dapit-tunnel
```

#### 4. **Set tujuan service**

Misalnya kamu mau expose web server lokal di port 80:

```bash
cloudflared tunnel route dns dapit-tunnel dapit.klandestin.site
```
#### 5. **Cek id-tunnel**

```bash
cd ~/.cloudflared/
```

#### 6. **Konfigurasi file tunnel**

Buat file config di `/etc/cloudflared/config.yml`

```yaml
tunnel: dapit-tunnel
credentials-file: /root/.cloudflared/<id-tunnel>.json

ingress:
  - hostname: dapit.klandestin.site
    service: http://localhost:80
  - service: http_status:404
```

#### 7. **Jalankan Tunnel**

```bash
cloudflared tunnel run dapit-tunnel
```

