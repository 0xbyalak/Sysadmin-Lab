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
zone "0xbyalak.local" {
    type master;
    file "/etc/bind/db.0xbyalak.local";
};

// Zona reverse
zone "10.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.10";
};
```
## Konfigurasi Forward zone
> Forward zone itu zona DNS yang mengarahkan domain → IP.
Buat & edit file:
```bash
cp /etc/bind/db.local /etc/bind/db.0xbyalak.local

nano /etc/bind/db.0xbyalak.local
```
Ubah & tambahkan:
```yaml
; BIND data file for 0xbyalak.local
$TTL    604800
@       IN      SOA     ns.0xbyalak.local. admin.0xbyalak.local. (
                             2         ; Serial (naikkan tiap ubah)
                        604800         ; Refresh
                         86400         ; Retry
                       2419200         ; Expire
                        604800 )       ; Negative Cache TTL

@       IN      NS      ns.kantor.local.
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
@       IN      SOA     ns.0xbyalak.local. admin.0xbyalak.local. (
                             2         ; Serial
                        604800         ; Refresh
                         86400         ; Retry
                       2419200         ; Expire
                        604800 )       ; Negative Cache TTL

@       IN      NS      ns.0xbyalak.local.
1       IN      PTR     ns.0xbyalak.local.
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
`dig @192.168.50.1 www.0xbyalak.local`

; <<>> DiG 9.18.30-0ubuntu0.24.04.2-Ubuntu <<>> @192.168.10.1 www.0xbyalak.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 59342
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.0xbyalak.local.            IN      A

;; AUTHORITY SECTION:
.                       10800   IN      SOA     a.root-servers.net. nstld.verisign-grs.com. 2025060301 1800 900 604800 86400

;; Query time: 101 msec
;; SERVER: 192.168.10.1#53(192.168.10.1) (UDP)
;; WHEN: Wed Jun 04 05:25:48 UTC 2025
;; MSG SIZE  rcvd: 122


