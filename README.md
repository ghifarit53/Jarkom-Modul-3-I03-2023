# Laporan Resmi Jarkom Modul 3 I03 2023

**Group I03** 
**Members:** 
- Fauzan Ahmad Faisal (5025211067)
- Muhammad Ghifari Taqiuddin (5025211063)

## No. 0
We need to create riegel.canyon.i03.com that points to Laravel worker with IP 10.60.4.1 (Frieren)
and granz.channel.i03.com that points to PHP worker with IP 10.60.3.1. (Lawine)
We can create the bind9 configuration in node Heiter

File: `named.conf.local`
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

File: `riegel.canyon.i03.com`
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

File: `granz.channel.i03.com`
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


## No. 2
## No. 3
## No. 4
## No. 5
## No. 6
## No. 7
## No. 8
## No. 9
## No. 10
## No. 11
## No. 12
