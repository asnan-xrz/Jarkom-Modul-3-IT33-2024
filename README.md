# **LAPORAN RESMI PRAKTIKUM KOMUNIKASI DATA & JARINGAN KOMPUTER MODUL 3**

|Nama                 |NRP               |                                   
|---------------------|----------|
|Agas Ananta Wijaya |5027221004|
---
Berikut adalah Laporan Resmi Praktikum Komunikasi Data & Jaringan Komputer Modul 3 oleh Kelompok IT33.

---

## Topologi

<img width="1280" alt="topologi" src="https://github.com/asnan-xrz/Jarkom-Modul-3-IT33-2024/assets/133721836/d2ec3501-aefb-4fa8-a349-bb82c6f37d13">

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

#### Pada `Arakis`

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

#### Pada `Irulan`

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
<img width="496" alt="tes ping" src="https://github.com/asnan-xrz/Jarkom-Modul-3-IT33-2024/assets/133721836/df9a75f4-d155-4f23-a471-d6ce84c99292">


## Nomor 2-5

Lakukan konfigurasi sesuai dengan peta yang sudah diberikan.Kemudian, karena masih banyak spice yang harus dikumpulkan, bantulah para aterides untuk bersaing dengan harkonen dengan kriteria berikut.:
1. Semua **CLIENT** harus menggunakan konfigurasi dari DHCP Server.
2. Client yang melalui House Harkonen mendapatkan range IP dari [prefix IP].1.14 - [prefix IP].1.28 dan [prefix IP].1.49 - [prefix IP].1.70 **(2)**
3. Client yang melalui House Atreides mendapatkan range IP dari [prefix IP].2.15 - [prefix IP].2.25 dan [prefix IP].2.200 - [prefix IP].4.210 **(3)**
4. Client mendapatkan DNS dari **Princess Irulan** dan dapat terhubung dengan internet melalui DNS tersebut **(4)**
5. Durasi DHCP server meminjamkan alamat IP kepada Client yang melalui House Harkonen selama 5 menit sedangkan pada client yang melalui House Atreides selama 20 menit. Dengan waktu maksimal dialokasikan untuk peminjaman alamat IP selama 87 menit **(5)**\
*house == switch

#### Pada `Mohiam`

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

#### Pada Client

<img width="1005" alt="tes udhcp client" src="https://github.com/asnan-xrz/Jarkom-Modul-3-IT33-2024/assets/133721836/770a1a8c-58ab-4fda-9f80-b60825824987">

## Nomor 6

Vladimir Harkonen memerintahkan setiap worker(harkonen) PHP, untuk melakukan konfigurasi virtual host untuk website berikut dengan menggunakan php 7.3.

#### Worker

```bash
apt-get update
apt-get install lynx -y
apt-get install wget -y
apt-get install unzip -y
apt-get install nginx -y
apt-get install php7.3 -y
apt-get install php7.3-fpm -y

mkdir -p /var/www/harkonen.it33.com

wget --no-check-certificate 'https://drive.google.com/uc?export=download&id=1lmnXJUbyx1JDt2OA5z_1dEowxozfkn30' -O /var/www/harkonen.it33.com.zip
unzip /var/www/harkonen.it33.com.zip -d /var/www/harkonen.it33.com
mv /var/www/harkonen.it33.com/modul-3 /var/www/harkonen.it33.com
rm -rf /var/www/harkonen.it33.com.zip

echo '
server {

        listen 80;

        root /var/www/harkonen.it33.com;

        index index.php index.html index.htm;
        server_name _;

        location / {
                        try_files $uri $uri/ /index.php?$query_string;
        }

        # pass PHP scripts to FastCGI server
        location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;
        }

location ~ /\.ht {
                        deny all;
        }

        error_log /var/log/nginx/jarkom_error.log;
        access_log /var/log/nginx/jarkom_access.log;
 }' > /etc/nginx/sites-available/harkonen.it33.com

echo '
<!DOCTYPE html>
<html>
<head>
    <title>House of Harkonen</title>
    <link rel="stylesheet" type="text/css" href="css/styles.css">
</head>
<body>
    <div class="container">
        <h1>This is Harkonen</h1>
        <p><?php
            $hostname = gethostname();
            echo "Request ini dihandle oleh: $hostname<br>"; ?> </p>
        <p>Enter your name to validate:</p>
        <form method="POST" action="index.php">
            <input type="text" name="name" id="nameInput">
            <button type="submit" id="submitButton">Submit</button>
        </form>
        <p id="greeting"><?php
            if(isset($_POST['name'])) {
                $name = $_POST['name'];
                echo "Hello, $name!";
            }
        ?></p>
    </div>

    <script src="js/script.js"></script>
</body>
</html>
' > /var/www/harkonen.it33.com/index.php

ln -s /etc/nginx/sites-available/harkonen.it33.com /etc/nginx/sites-enabled
rm -rf /etc/nginx/sites-enabled/default

service php7.3-fpm start
service php7.3-fpm restart
service nginx restart
nginx -t
```

#### Stilgar (Load Balancer)

```bash
apt-get update
apt-get install bind9 nginx -y

echo '
 upstream myweb  {
        server 10.80.1.4; #IP Vladimir
        server 10.80.1.5; #IP Rabban
        server 10.80.1.3; #IP Feyd
 }

 server {
        listen 80;
        server_name harkonen33.com;

        location / {
        proxy_pass http://myweb;
        }
 }' > /etc/nginx/sites-available/lb-jarkom

ln -s /etc/nginx/sites-available/lb-jarkom /etc/nginx/sites-enabled
rm -rf /etc/nginx/sites-enabled/default

service nginx restart
nginx -t
```

## Nomor 7

Aturlah agar Stilgar dari fremen dapat dapat bekerja sama dengan maksimal, lalu lakukan testing dengan 5000 request dan 150 request/second

#### Pada CLient

```
apt-get update
apt-get install apache2-utils -y
ab -V
ab -n 5000 -c 150 http://harkonen.it33.com/
```

## Nomor 8

Karena diminta untuk menuliskan peta tercepat menuju spice, buatlah analisis hasil testing dengan 500 request dan 50 request/second masing-masing algoritma Load Balancer dengan ketentuan sebagai berikut:\
a. Nama Algoritma Load Balancer\
b. Report hasil testing pada Apache Benchmark
c. Grafik request per second untuk masing masing algoritma. 
d. Analisis

#### Pada Client

Jalankan `ab -n 500 -c 50 http://harkonen.it33.com/`

1. Round Robin
```
Benchmarking harkonen.it33.com (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Finished 500 requests


Server Software:        nginx/1.14.2
Server Hostname:        harkonen.it33.com
Server Port:            80

Document Path:          /
Document Length:        602 bytes

Concurrency Level:      50
Time taken for tests:   0.738 seconds
Complete requests:      500
Failed requests:        333
   (Connect: 0, Receive: 0, Length: 333, Exceptions: 0)
Total transferred:      368502 bytes
HTML transferred:       300002 bytes
Requests per second:    677.67 [#/sec] (mean)
Time per request:       73.782 [ms] (mean)
Time per request:       1.476 [ms] (mean, across all concurrent requests)
Transfer rate:          487.74 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   17   5.1     16      36
Processing:     3   55  10.3     55      84
Waiting:        2   55  10.3     55      84
Total:          3   72  11.0     72      97

Percentage of the requests served within a certain time (ms)
  50%     72
  66%     74
  75%     76
  80%     78
  90%     85
  95%     95
  98%     97
  99%     97
 100%     97 (longest request)
```

2. Least-Connection
```
Benchmarking harkonen.it33.com (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Finished 500 requests


Server Software:        nginx/1.14.2
Server Hostname:        harkonen.it33.com
Server Port:            80

Document Path:          /
Document Length:        602 bytes

Concurrency Level:      50
Time taken for tests:   0.617 seconds
Complete requests:      500
Failed requests:        334
   (Connect: 0, Receive: 0, Length: 334, Exceptions: 0)
Total transferred:      368498 bytes
HTML transferred:       299998 bytes
Requests per second:    810.80 [#/sec] (mean)
Time per request:       61.668 [ms] (mean)
Time per request:       1.233 [ms] (mean, across all concurrent requests)
Transfer rate:          583.55 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        1   13   5.4     13      28
Processing:     3   47   6.4     46      62
Waiting:        2   47   6.5     46      62
Total:          4   60   8.1     62      73

Percentage of the requests served within a certain time (ms)
  50%     62
  66%     64
  75%     66
  80%     67
  90%     68
  95%     69
  98%     70
  99%     71
 100%     73 (longest request)
```

3. IP Hash
```
Benchmarking harkonen.it33.com (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Finished 500 requests


Server Software:        nginx/1.14.2
Server Hostname:        harkonen.it33.com
Server Port:            80

Document Path:          /
Document Length:        598 bytes

Concurrency Level:      50
Time taken for tests:   0.637 seconds
Complete requests:      500
Failed requests:        0
Total transferred:      367500 bytes
HTML transferred:       299000 bytes
Requests per second:    785.20 [#/sec] (mean)
Time per request:       63.678 [ms] (mean)
Time per request:       1.274 [ms] (mean, across all concurrent requests)
Transfer rate:          563.59 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   14   5.0     16      25
Processing:     2   48   7.7     49      61
Waiting:        1   48   7.7     48      61
Total:          3   62  10.0     62      78

Percentage of the requests served within a certain time (ms)
  50%     62
  66%     67
  75%     68
  80%     70
  90%     73
  95%     75
  98%     77
  99%     77
 100%     78 (longest request)
```

4. Generic Hash
```
Benchmarking harkonen.it33.com (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Finished 500 requests


Server Software:        nginx/1.14.2
Server Hostname:        harkonen.it33.com
Server Port:            80

Document Path:          /
Document Length:        600 bytes

Concurrency Level:      50
Time taken for tests:   1.335 seconds
Complete requests:      500
Failed requests:        0
Total transferred:      368500 bytes
HTML transferred:       300000 bytes
Requests per second:    374.43 [#/sec] (mean)
Time per request:       133.538 [ms] (mean)
Time per request:       2.671 [ms] (mean, across all concurrent requests)
Transfer rate:          269.48 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   13   5.2     13      25
Processing:     2   45  43.4     43    1002
Waiting:        1   45  43.4     43    1002
Total:          2   58  44.1     58    1021

Percentage of the requests served within a certain time (ms)
  50%     58
  66%     59
  75%     61
  80%     62
  90%     66
  95%     67
  98%     68
  99%     68
 100%   1021 (longest request)
```
