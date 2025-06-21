# SAMBA
Samba adalah perangkat lunak open‑source di Linux/Unix yang “menirukan” (meng­implementasi) protokol SMB/CIFS‑­nya Windows. Hasilnya, server Linux bisa tampil di jaringan seperti layaknya “File & Printer Sharing” pada Windows—lengkap dengan akses folder bersama (share), map network drive, sampai autentikasi domain.

## Instalasi
```bash
apt update && apt install samba -y
```

## Buat directory sharing
```bash
mkdir -p /srv/samba/share
chown root:staff /srv/samba/share
chmod 770 /srv/samba/share
```

## Tambahkan user dan grup khusus
```bash
groupadd staff
useradd -M -s /sbin/nologin 0xbyalak
usermod -aG staff 0xbyalak
smbpasswd -a 0xbyalak
```
Penjelasan:

`-M`: tidak buat home.

`-s /sbin/nologin`: agar user tidak bisa login shell.

`smbpasswd`: untuk set password Samba.

## Konfigurasi
> Edit konfigurasi utama Samba agar hanya user tertentu yang bisa akses, dan trafik terenkripsi.

Backup & Buka file konfigurasi:
```bash
cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
nano /etc/samba/smb.conf
```

Ubah dan tambahkan:
```bash
[global]
   server string = File sharing SAMBA server    # Nama server yang tampil di jaringan
   workgroup = WORKGROUP                        # Nama workgroup Windows (default: WORKGROUP)
   security = user                              # Autentikasi berdasarkan user Samba
   map to guest = never                         # Tolak akses guest, wajib login user valid
   smb encrypt = required                       # Wajibkan enkripsi SMB (SMB3) untuk keamanan
   server signing = mandatory                   # Aktifkan signing untuk integritas data
   log file = /var/log/samba/%m.log             # Simpan log per client (berdasarkan NetBIOS name)
   log level = 2                                # Tingkat log detail (cukup untuk audit ringan)
   max log size = 1000                          # Maks ukuran log 1000 KB (1 MB)
   interfaces = lo eth0                         # Hanya dengarkan di loopback dan eth0
   bind interfaces only = yes                   # Jangan dengarkan di interface lain
   disable netbios = yes                        # Nonaktifkan NetBIOS (port 137/138)
   smb ports = 445                              # Hanya gunakan port modern (445), tanpa 139
   client min protocol = SMB2                   # Paksa klien menggunakan minimal protokol SMB2 (lebih aman dari SMB1)
   client max protocol = SMB3                   # Batasi maksimal protokol SMB ke versi SMB3 (versi terbaru & terenkripsi)

[share]
   path = /srv/samba/share                      # Direktori yang akan dishare
   valid users = @staff                         # Hanya user dalam grup 'staff' yang boleh akses
   read only = no                               # Akses full (bisa baca dan tulis)
   browsable = no                               # Tidak tampil di 'Network', harus diakses manual
   guest ok = no                                # Tidak izinkan akses guest sama sekali
   create mask = 0770                           # File baru: hanya bisa diakses owner & grup
   directory mask = 0770                        # Folder baru: sama, hanya owner & grup
   vfs objects = full_audit                     # Aktifkan modul audit (logging aktivitas file)
   full_audit:prefix = %u|%I|%S                 # Format log: user|IP|share_name
```
> Hilangkan komentar lebih dulu
## Restart service
```bash
systemctl restart smbd nmbd
systemctl enable --now smbd
systemctl status smbd
```

Cek konfigurasi:
```bash
testparm
```

## Konfigurasi Firewall
> Ijinkan hanya dari jaringan internal

```bash
ufw allow from 192.168.1.0/24 to any port 445 proto tcp
ufw deny in to any port 445
ufw reload
```

## Install & Konfigurasi fail2ban (Cegah Brute Force)

```bash
sudo apt install fail2ban
```

### Buat filter:

```bash
sudo nano /etc/fail2ban/filter.d/samba-auth.conf
```

```ini
[Definition]
failregex = .*Authentication for user \[.*\] -> \[.*\] FAILED.*
```

### Buat jail:

```bash
sudo nano /etc/fail2ban/jail.d/samba.conf
```

```ini
[samba-auth]
enabled = true
port = 445
filter = samba-auth
logpath = /var/log/samba/*.log
maxretry = 5
findtime = 600
bantime = 3600
```

```bash
sudo systemctl restart fail2ban
```

## Pengujian
Dari windows:
```bash

