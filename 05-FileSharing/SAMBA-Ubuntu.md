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
cp /etc/samba/smb.conf /etc/samba/smb.conf.backup
truncate -s 0 /etc/samba/smb.conf
nano /etc/samba/smb.conf
```

Ubah dan tambahkan:
```bash
[global]
   server string = Samba Server
   workgroup = WORKGROUP

   security = user                     # User-based authentication
   map to guest = never                # Tolak akses guest sepenuhnya

   smb encrypt = required              # Wajib enkripsi (SMB3)
   server signing = mandatory          # Wajib signing (data integrity)

   log file = /var/log/samba/%m.log
   log level = 2                       # Cukup untuk audit ringan
   max log size = 1000

   interfaces = lo eth0                # Batasi interface
   bind interfaces only = yes

   disable netbios = yes               # Nonaktifkan NetBIOS
   smb ports = 445                     # Hanya port 445

   client min protocol = SMB2          # Minimal SMB2 (disable SMB1)
   client max protocol = SMB3          # Maksimal SMB3
   server min protocol = SMB2
   server max protocol = SMB3

   passdb backend = tdbsam             # Backend default, bisa ganti ke LDAP/AD jika perlu
   name resolve order = host wins bcast

   load printers = no                  # Non-printer server
   printing = bsd
   printcap name = /dev/null

[share]
   path = /srv/samba/share
   valid users = @staff
   read only = no
   create mask = 0770
   directory mask = 0770
   guest ok = no
   browsable = no

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
apt install fail2ban -y
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
Dari Linux:
```bash
root@klandestin:/srv/samba# smbclient //192.168.0.101/sambashare -U 0xbyalak

Password for [WORKGROUP\0xbyalak]:
Try "help" to get a list of possible commands.
smb: \>
```

Dari Windows:

![Screenshot 2025-06-21 232939](https://github.com/user-attachments/assets/939b3360-be00-4052-96bf-1a3d3ee40f7a)

![Screenshot 2025-06-23 123819](https://github.com/user-attachments/assets/ef63f1be-ad51-4107-bbe4-59bdec5c580a)

![Screenshot 2025-06-23 141446](https://github.com/user-attachments/assets/7858c66b-a2b3-423a-b8d2-e67bdc821e16)

