# Laporan Resmi Jarkom Modul 3 I03 2023

**Group I03** 
**Members:** 
- Fauzan Ahmad Faisal (5025211067)
- Muhammad Ghifari Taqiuddin (5025211063)

## No. 0
We need to create riegel.canyon.i03.com that points to Laravel worker with IP 10.60.4.1 (Frieren)
and granz.channel.i03.com that points to PHP worker with IP 10.60.3.1. (Lawine)
We can create the bind9 configuration in node Heiter

File: `/etc/bind/named.conf.local`
```
zone "riegel.canyon.i03.com" {
        type master;
        file "/etc/bind/jarkom/riegel.canyon.i03.com";
};

zone "granz.channel.i03.com" {
        type master;
        file "/etc/bind/jarkom/granz.channel.i03.com";
};
```

File: `/etc/bind/jarkom/riegel.canyon.i03.com`
```
;
; riegel.canyon.i03.com
; Laravel Worker
;
$TTL    604800
@       IN      SOA     riegel.canyon.i03.com. root.riegel.canyon.i03.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      localhost.
@       IN      A       10.60.4.1 ; Frieren's IP Address
```

File: `/etc/bind/jarkom/granz.channel.i03.com`
```
;
; granz.channel.i03.com
; PHP Worker
;
$TTL    604800
@       IN      SOA     granz.channel.i03.com. root.granz.channel.i03.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      localhost.
@       IN      A       10.60.3.1 ; Lawine's IP Address
```

Restart the bind9 service
```
service bind9 restart
```

## No. 1
Topology

<img width="798" alt="topology-i03" src="https://user-images.githubusercontent.com/59758342/283246078-d9c1be3b-921c-4d16-87a1-7c5ded059484.png">

## No. 2, 3, and 5
Clients that connects through Switch3, Revolte and Richter, get the suffix 3.16-3.32 and 3.64.
Clients that connects through Switch4, Sein and Stark get the suffix 4.12-4.20 and 4.160-4.168.


First, we need to install `isc-dhcp-server` in node Himmel.
Then, put these configurations into their respective path

We set the INTERFACESv4 to eth0 since we are connected through it

File: `/etc/default/isc-dhcp-server`
```conf
# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACESv4="eth0"
# INTERFACESv6=""
```

Then, we set the IP range, gateway, broadcast address,
DNS server IP, and least time according to the requirements

File: `/etc/dhcp/dhcpd.conf`
```conf
subnet 10.60.1.0 netmask 255.255.255.0 {}

subnet 10.60.3.0 netmask 255.255.255.0 {
    range 10.60.3.16 10.60.3.32;
    range 10.60.3.64 10.60.3.80;
    option routers 10.60.3.10;
    option broadcast-address 10.60.3.255;
    option domain-name-servers 10.60.1.2;
    default-lease-time 180; # 3 Menit
    max-lease-time 5760; # 96 Menit
}

subnet 10.60.4.0 netmask 255.255.255.0 {
    range 10.60.4.12 10.60.4.20;
    range 10.60.4.160 10.60.4.168;
    option routers 10.60.4.10;
    option broadcast-address 10.60.4.255;
    option domain-name-servers 10.60.1.2;
    default-lease-time 720; # 12 Menit
    max-lease-time 5760; # 96 Menit
}
```

## No. 4
## No. 6
## No. 7
## No. 8
## No. 9
## No. 10
## No. 11
## No. 12
