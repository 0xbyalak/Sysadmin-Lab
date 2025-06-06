# BIND9
> BIND9 (kepanjangan dari Berkeley Internet Name Domain versi 9) adalah DNS server software yang paling populer dan paling sering dipakai buat bikin DNS server sendiri, baik di jaringan lokal (intranet) maupun publik (internet).
## Install bind9
> Install bind9 dan tools tambahan.
```bash
apt update
sudo apt install bind9 bind9utils -y
```
## Konfigurasi file Zona `named.conf.local`
Edit file:
```bash
sudo nano /etc/bind/named.conf.local
```
Tambahkan:
```yaml
// Zona domain forward
zone "0xbyalak.com" {
    type master;
    file "/etc/bind/db.0xbyalak.com";
};

// Zona reverse
zone "50.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.50";
};
```
## Konfigurasi Forward zone
> Forward zone itu zona DNS yang mengarahkan domain → IP.
Buat & edit file:
```bash
cp /etc/bind/db.local /etc/bind/db.0xbyalak.com

nano /etc/bind/db.0xbyalak.com
```
Ubah & tambahkan:
```yaml
; BIND data file for 0xbyalak.com
$TTL    604800
@       IN      SOA     ns.0xbyalak.com. admin.0xbyalak.com. (
                             2         ; Serial
                        604800         ; Refresh
                         86400         ; Retry
                       2419200         ; Expire
                        604800 )       ; Negative Cache TTL

@       IN      NS      ns.0xbyalak.com.
ns      IN      A       192.168.50.1

www     IN      A       192.168.50.1
mail    IN      A       192.168.50.1
ftp     IN      A       192.168.50.1
login   IN      A       192.168.50.1
```
## Konfigurasi reverse zone
> Reverse zone itu zona DNS yang mengarahkan IP → nama domain. Biasanya buat keperluan validasi (email, logging, monitoring).
Buat dan edit file:
```bash
cp /etc/bind/db.127 /etc/bind/db.192.168.50

nano /etc/bind/db.192.168.50
```
Ubah & tambahkan:
```yaml
;
; Reverse zone for 192.168.50.0/24
;
$TTL    604800
@       IN      SOA     ns.0xbyalak.com. admin.0xbyalak.com. (
                             2         ; Serial
                        604800         ; Refresh
                         86400         ; Retry
                       2419200         ; Expire
                        604800 )       ; Negative Cache TTL

@       IN      NS      ns.0xbyalak.com.
1       IN      PTR     ns.0xbyalak.com.
```
## Konfigurasi IP resolver local
Edit file:
`nano /etc/resolv.conf`
Ubah:
```bash
nameserver 192.168.50.1
search 0xbyalak.local
```
`save & keluar`
## Cek konfigurasi & restart
Cek `named-checkconf`
Note: Jika tidak ada error maka setup selesai.
Restart service:
```bash
systemctl restart bind9
systemctl enable --now bind9
```
## Testing
Tulis perintah:

`dig @192.168.50.1 www.0xbyalak.local`

jika berhasil:
```bash
; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> @192.168.50.1 www.0xbyalak.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34656
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: b94ce73e4d5fd178010000006842857b0e2cee9ea26f9d7b (good)
;; QUESTION SECTION:
;www.0xbyalak.com.              IN      A

;; ANSWER SECTION:
www.0xbyalak.com.       604800  IN      A       192.168.50.1

;; Query time: 0 msec
;; SERVER: 192.168.50.1#53(192.168.50.1) (UDP)
;; WHEN: Fri Jun 06 06:06:51 UTC 2025
;; MSG SIZE  rcvd: 89
```


