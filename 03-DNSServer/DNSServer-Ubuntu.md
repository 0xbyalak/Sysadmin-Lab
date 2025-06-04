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
## 
