# **LAPORAN RESMI PRAKTIKUM KOMUNIKASI DATA & JARINGAN KOMPUTER MODUL 3**

|Nama                 |NRP               |                                   
|---------------------|----------|
|Agas Ananta Wijaya |5027221004|
---
Berikut adalah Laporan Resmi Praktikum Komunikasi Data & Jaringan Komputer Modul 3 oleh Kelompok IT33.

---

## Topologi

(screenshot)

## Konfigurasi
Lakukan konfigurasi sesuai dengan peta yang sudah diberikan.

### Arakis (DHCP Relay)

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.80.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.80.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 10.80.3.1
	netmask 255.255.255.0

auto eth4
iface eth4 inet static
	address 10.80.4.1
	netmask 255.255.255.0
```

### Mohiam (DHCP)

```
auto eth0
iface eth0 inet static
	address 10.80.3.3
	netmask 255.255.255.0
	gateway 10.80.3.1
```

### Irulan (DNS)

```
auto eth0
iface eth0 inet static
	address 10.80.3.4
	netmask 255.255.255.0
	gateway 10.80.3.1
```

### Harkonen (Switch 1)

**Vladimir (PHP Worker)**

```
auto eth0
iface eth0 inet static
	address 10.80.1.4
	netmask 255.255.255.0
	gateway 10.80.1.1
```

**Rabban (PHP Worker)**

```
auto eth0
iface eth0 inet static
	address 10.80.1.5
	netmask 255.255.255.0
	gateway 10.80.1.1
```


**Feyd (PHP Worker)**

```
auto eth0
iface eth0 inet static
	address 10.80.1.3
	netmask 255.255.255.0
	gateway 10.80.1.1
```

### Atreides (Switch 2)

**Leto (Laravel Worker)**

```
auto eth0
iface eth0 inet static
	address 10.80.2.4
	netmask 255.255.255.0
	gateway 10.80.2.1
```

**Duncan (Laravel Worker)**

```
auto eth0
iface eth0 inet static
	address 10.80.2.5
	netmask 255.255.255.0
	gateway 10.80.2.1
```

**Jessica (Laravel Worker)**

```
auto eth0
iface eth0 inet static
	address 10.80.2.3
	netmask 255.255.255.0
	gateway 10.80.2.1
```

### Fremen

**Chani (Database Server)**

```
auto eth0
iface eth0 inet static
	address 10.80.4.5
	netmask 255.255.255.0
	gateway 10.80.4.1
```

**Stilgar (Load Balancer)**

```
auto eth0
iface eth0 inet static
	address 10.80.4.4
	netmask 255.255.255.0
	gateway 10.80.4.1
```

### Dmitri dan Paul (Client)

```
auto eth0
iface eth0 inet dhcp
```

---

## Nomor 0

Planet Caladan sedang mengalami krisis karena kehabisan spice, klan atreides berencana untuk melakukan eksplorasi ke planet arakis dipimpin oleh duke leto mereka meregister domain name atreides.it33.com untuk worker Laravel mengarah pada Leto Atreides . Namun ternyata tidak hanya klan atreides yang berusaha melakukan eksplorasi, Klan harkonen sudah mendaftarkan domain name harkonen.it33.com untuk worker PHP (0) mengarah pada Vladimir Harkonen

## Pada `Arakis`

```bash
#!/bin/bash


apt update
apt-get install isc-dhcp-relay -y
# Konfigurasi yang akan dimasukkan ke dalam file
echo "# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts


#
# This is a POSIX shell fragment
#


# What servers should the DHCP relay forward requests to?
SERVERS=\"10.80.3.3\"


# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES=\"eth1 eth2 eth3 eth4\"


# Additional options that are passed to the DHCP relay daemon?
OPTIONS=\"\"" >/etc/default/isc-dhcp-relay


sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/g' /etc/sysctl.conf


iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.80.0.0/16


service isc-dhcp-relay restart
service isc-dhcp-relay status
```

## Pada `Irulan`

```bash
#!/bin/bash

apt-get update
apt-get install bind9 -y

forward="options {
directory \"/var/cache/bind\";
forwarders {
  	   192.168.122.1;
};

allow-query{any;};
listen-on-v6 { any; };
};
"
echo "$forward" > /etc/bind/named.conf.options

echo "zone \"atreides.it33.com\" {
	type master;
	file \"/etc/bind/jarkom/atreides.it33.com\";
};

zone \"harkonen.it33.com\" {
	type master;
	file \"/etc/bind/jarkom/harkonen.it33.com\";
};
" > /etc/bind/named.conf.local

mkdir /etc/bind/jarkom

atreides="
;
;BIND data file for local loopback interface
;
\$TTL    604800
@    IN    SOA    atreides.it33.com. root.atreides.it33.com. (
        2        ; Serial
                604800        ; Refresh
                86400        ; Retry
                2419200        ; Expire
                604800 )    ; Negative Cache TTL
;                   
@    IN    NS    atreides.it33.com.
@       IN    A    10.80.2.4 ; IP Leto
"
echo "$atreides" > /etc/bind/jarkom/atreides.it33.com

harkonen="
;
;BIND data file for local loopback interface
;
\$TTL    604800
@    IN    SOA    harkonen.it33.com. root.harkonen.it33.com. (
        2        ; Serial
                604800        ; Refresh
                86400        ; Retry
                2419200        ; Expire
                604800 )    ; Negative Cache TTL
;                   
@    IN    NS    harkonen.it33.com.
@       IN    A    10.80.1.4 ; IP Vladimir
"
echo "$harkonen" > /etc/bind/jarkom/harkonen.it33.com

service bind9 restart
service bind9 status
```
(screenshot)

## Nomor 2-5

Lakukan konfigurasi sesuai dengan peta yang sudah diberikan.Kemudian, karena masih banyak spice yang harus dikumpulkan, bantulah para aterides untuk bersaing dengan harkonen dengan kriteria berikut.:
1. Semua **CLIENT** harus menggunakan konfigurasi dari DHCP Server.
2. Client yang melalui House Harkonen mendapatkan range IP dari [prefix IP].1.14 - [prefix IP].1.28 dan [prefix IP].1.49 - [prefix IP].1.70 **(2)**
3. Client yang melalui House Atreides mendapatkan range IP dari [prefix IP].2.15 - [prefix IP].2.25 dan [prefix IP].2.200 - [prefix IP].4.210 **(3)**
4. Client mendapatkan DNS dari **Princess Irulan** dan dapat terhubung dengan internet melalui DNS tersebut **(4)**
5. Durasi DHCP server meminjamkan alamat IP kepada Client yang melalui House Harkonen selama 5 menit sedangkan pada client yang melalui House Atreides selama 20 menit. Dengan waktu maksimal dialokasikan untuk peminjaman alamat IP selama 87 menit **(5)**\
*house == switch

## Pada `Mohiam`

```bash
#!/bin/bash

# Update package list and install isc-dhcp-server
apt update
apt install isc-dhcp-server -y

# Write the DHCP server configuration
echo "# Defaults for isc-dhcp-server (sourced by /etc/init.d/isc-dhcp-server)
# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
#DHCPDv4_CONF=/etc/dhcp/dhcpd.conf
#DHCPDv6_CONF=/etc/dhcp/dhcpd6.conf
# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPDv4_PID=/var/run/dhcpd.pid
#DHCPDv6_PID=/var/run/dhcpd6.pid
# Additional options to start dhcpd with.
#       Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS=\"\"
# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. \"eth0 eth1\".
INTERFACESv4=\"eth0\"
INTERFACESv6=\"\"" >/etc/default/isc-dhcp-server

# Corrected DHCP configuration
echo "subnet 10.80.1.0 netmask 255.255.255.0 {
    range 10.80.1.14 10.80.1.28;
    range 10.80.1.49 10.80.1.70;
    option routers 10.80.1.254;
    option broadcast-address 10.80.1.255;
    option domain-name-servers 10.80.3.4; # IP Irulan
    default-lease-time 300;
    max-lease-time 5220;
}

subnet 10.80.2.0 netmask 255.255.255.0 {
    range 10.80.2.15 10.80.2.25;
    range 10.80.2.200 10.80.2.210;
    option routers 10.80.2.254;
    option broadcast-address 10.80.2.255;
    option domain-name-servers 10.80.3.4; # IP Irulan
    default-lease-time 1200;
    max-lease-time 5220;
}

subnet 10.80.3.0 netmask 255.255.255.0 {
    option routers 10.80.3.254;
    option broadcast-address 10.80.3.255;
}

subnet 10.80.4.0 netmask 255.255.255.0 {
    option routers 10.80.4.254;
    option broadcast-address 10.80.4.255;
}" >/etc/dhcp/dhcpd.conf

# Remove PID file if it exists
rm /var/run/dhcpd.pid

# Restart and check the status of the DHCP server
service isc-dhcp-server restart
service isc-dhcp-server status
```

## Pada Client `Dmitri` dan `Paul`

(screenshot)

## Nomor 6

Vladimir Harkonen memerintahkan setiap worker(harkonen) PHP, untuk melakukan konfigurasi virtual host untuk website berikut dengan menggunakan php 7.3.

