# DHCP Server di VirtualBox
### Pilihan Interface di VirtualBox:
| Interface             | Bisa komunikasi VM ↔ Host? | Bisa laptop dapet IP dari VM DHCP? | Catatan                                                      |
| --------------------- | -------------------------- | ---------------------------------- | ------------------------------------------------------------ |
| **NAT**               |            ❌              |               ❌                  | Internet only, gak bisa interaksi langsung                   |
| **Bridged Adapter**   |             ✅             |               ✅                  | VM = seolah-olah device di jaringan yang sama                |
| **Host-Only Adapter** |             ✅             |               ✅                  | VM & host dalam jaringan tertutup, bagus buat simulasi lokal |
| **Internal Network**  |             ❌             |               ❌                  | Cuma antar VM (gak bisa ke host/laptop)                      |
## Aktifkan Adapter 2 di VM
![Screenshot 2025-06-03 220205](https://github.com/user-attachments/assets/fe146287-6718-4cc9-921e-12b6cd0b32f6)

## Konfigurasi Netwrok
> Skenarionya:

| Interface | Fungsi           | IP Address           |
| --------- | ---------------- | -------------------- |
| enp0s3    | Internet         | DHCP                 |
| enp0s8    | LAN / DHCP scope | Static: 192.168.50.1 |

### Edit file `netplan`
> Nama file menyesuaikan yang ada di direktory `/etc/netplan`

` nano /etc/netplan/50-cloud-init.yaml`
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.50.1/24
```
`Simpan & Keluar`
### Apply netplan & cek ip
```bash
netplan apply
```
```bash
ip a
```
`Output`:
```yaml
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:3e:5f:27 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.9/24 metric 100 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 86398sec preferred_lft 86398sec
    inet6 fe80::a00:27ff:fe3e:5f27/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:f4:f2:31 brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.1/24 brd 192.168.50.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fef4:f231/64 scope link
       valid_lft forever preferred_lft forever
```
## Install `isc-dhcp-server`
```bash
apt update
apt install isc-dhcp-server -y
```
## Set interface DHCP
`nano /etc/default/isc-dhcp-server`

```bash
INTERFACESv4="enp0s8"
INTERFACESv6=""
```
## Konfigurasi file utama
`Backup file dhcp.conf`
```bash
cp /etc/dhcp/dhcp.conf /etc/dhcp/dhcp.conf.backup
```
`Cari dan ubah/tambahkan `:
```bash
nano /etc/dhcp/dhcp.conf
```
```yaml
# === Info global ===
authoritative;                          # DHCP server ini yang pegang kendali IP
log-facility local7;                    # Logging ke syslog local7

# === Subnet utama ===
subnet 192.168.10.0 netmask 255.255.255.0 {
  range 192.168.10.100 192.168.10.200;   # Rentang IP yang dibagikan
  option routers 192.168.10.1;           # Gateway (biasanya IP router/firewall)
  option subnet-mask 255.255.255.0;      # Netmask
  option domain-name "0xbyalak.net.local";     # Domain internal
  option domain-name-servers 8.8.8.8, 1.1.1.1;  # DNS
}

# === Optional : Static Reservation (MAC Binding) ===
host printer1 {
  hardware ethernet 08:00:27:aa:bb:cc;   # MAC Address
  fixed-address 192.168.10.50;           # IP tetap
}
```
### Aktifkan service
```bash
systemctl enable --now isc-dhcp-server
systemctl start isc-dhcp-server
```
### Cek koneksi
lihat ip laptop:

![Screenshot 2025-06-04 001531](https://github.com/user-attachments/assets/8fd47b81-0d7f-4113-84fd-0beeed3c3ce0)
