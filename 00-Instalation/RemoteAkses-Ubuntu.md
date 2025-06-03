# SSH (Secure Shell)
> SSH (Secure Shell) adalah protokol jaringan yang dipakai buat akses dan kontrol server dari jarak jauh, dengan aman lewat jalur yang terenkripsi.
> Kita bisa ngendaliin server dari terminal, seolah-olah kita lagi duduk langsung depan servernya — padahal kita aksesnya dari laptop kita, bisa dari rumah, kafe, kampus, atau mana aja.
## Install openssh-server
Kalau kita pakai Ubuntu Server, biasanya udah include. Tapi kalau belum, install manual:
#### Install:
`Note: Pastikan sudah mempunyai akses root`
```bash
apt update
apt install openssh-server -y
```
#### Cek status:
```bash
systemctl status ssh
```
Kalau belum aktif, aktifkan lebih dulu.
#### Aktifkan:
```
systemctl enable --now ssh
```
#### Tes Koneksi
Dari terminal laptop `ssh username@ipaddress`:
```
ssh 0xbyalak@192.168.1.1
```
Kalau berhasil masuk, berarti service sudah ready.
## Hardening
>File konfigurasi ssh ada didalam direktory `/etc/ssh/sshd_config`.
### Backup konfigurasi default:
```bash
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```
### Edit file `sshd_config`:
>Masuk file menggunakan teks editor:
```bash
nano /etc/ssh/sshd_config
```
### Isi file Konfigurasi:
> Cari dan ubah/tambahkan:
```yaml
# === Port & Protocol ===
Port 2025                            # Ganti port default
Protocol 2                           # Pakai protocol versi 2

# === User Access Control ===
PermitRootLogin no                  # Disable login langsung sebagai root
AllowUsers 0xbyalak dapit           # Hanya user tertentu yang boleh akses lewat SSH
AllowGroups klandestin              # Mengijinkan group akses
Match User dapit                    # Bikin aturan yang berlaku hanya untuk user tertentu
        PasswordAuthentication yes
Match Groups klandestin             # Bikin aturan yang berlaku hanya untuk group tertentu
        PasswordAuthentication yes

# === Authentication ===
PasswordAuthentication no           # Disable login pakai password, hanya pakai SSH key
PubkeyAuthentication yes            # Aktifkan autentikasi pakai SSH key
ChallengeResponseAuthentication no  # Disable metode challenge (misal keyboard-interactive)
UsePAM yes                          # Tetap aktifkan PAM untuk integrasi sistem

# === Session Management ===
LoginGraceTime 30                   # Waktu tunggu 30 detik sebelum sesi dibatalkan
ClientAliveInterval 300             # Kirim sinyal keep-alive setiap 5 menit
ClientAliveCountMax 2               # Maks 2x gagal respon client sebelum disconnect otomatis
MaxAuthTries 3                      # Maks 3x percobaan login sebelum koneksi ditolak

# === Logging ===
LogLevel VERBOSE                    # Logging verbose, tampilkan user & IP setiap login
SyslogFacility AUTH                 # Kirim log ke facility AUTH

# === X11 & Tunneling ===
X11Forwarding no                    # Matikan X11 forwarding (nggak butuh di server headless)
AllowTcpForwarding no               # Matikan port forwarding (kecuali kalau butuh)
PermitTunnel no                     # Disable VPN-style tunneling lewat SSH

# === Key & File Location ===
AuthorizedKeysFile .ssh/authorized_keys  # File public key client

# === Banner ===
Banner /etc/issue.net               # Tampilkan peringatan legal sebelum login
```
`Save & keluar`
### Bikin Banner
Buat file banner `nano /etc/issue.net`:
```bash
WARNING: Unauthorized access to this system is prohibited.
All activities are logged and monitored.
```
`Save & keluar`
### Buat ssh-key
` Di terminal laptop atau komputer `:
```bash
ssh-keygen -t ed25519 -C "akses 0xbyalak"
```
> Ini adalah perintah buat membuat SSH key pair (public key & private key) dengan algoritma ed25519, dan kasih komentar biar gampang dikenali.
Hasilnya:
> - ~/.ssh/id_rsa ➜ private key (jangan dibagikan)
> - ~/.ssh/id_rsa.pub ➜ public key
### Uploade public key ke server
```bash
ssh-copy-id 0xbyalak@192.168.1.1
```
Pastikan file sudah ada di direktory `~/.ssh/authorized_keys`
### Restart service
> Kembali ke server
```bash
systemctl restart ssh
```
### Cek koneksi
```bash
ssh 0xbyalak@192.168.1.1 -p 2025
```
Jika berhasil masuk, berarti setup sudah berhasil.






