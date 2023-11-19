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

Then, restart the `bind9` service

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

Then, restart `isc-dhcp-server`

Then, we set up a DNS relay in node Aura so the DHCP server
can distribute the IP to all the clients

File: `/etc/default/isc-dhcp-relay`
```conf
# Defaults for isc-dhcp-relay initscript
# sourced by /etc/init.d/isc-dhcp-relay
# installed at /etc/default/isc-dhcp-relay by the maintainer scripts

#
# This is a POSIX shell fragment
#

# What servers should the DHCP relay forward requests to?
SERVERS="10.60.1.1"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="eth1 eth2 eth3 eth4"

# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```

Then restart the `isc-dhcp-relay`

## No. 4
All the clients will get internet access from the DNS Server Heiter.
First we set up a forwarder there. We already set the DNS server
in `dhcpd.conf`, now we can set the configurations for DNS
server to be able to forward internet connection

File: `/etc/bind/named.conf.options`
```sh
options {
	directory "/var/cache/bind";

	forwarders {
		192.168.122.1;
	};

	listen-on-v6 { any; };
	allow-query{ any; };
};
```

Then, restart `bind9` service. When we check
the `/etc/resolv.conf` on each client node, they should
point at Heiter's IP address

## No. 6

We need to install `wget` and `unzip` in order to download
the website assets from Google Drive. And then `nginx`, `php`, and
`php-fpm` to make the site online.

```sh
apt update
apt install wget unzip nginx php php-fpm -y
```

Then, we download the zip file from Google Drive, unzip it, then
move it to `/var/www` directory. Then, create a configuration file
for the website, and put it in `/etc/nginx/sites-available/jarkom`

File: `/etc/nginx/sites-available/jarkom`
```conf
server {
        listen 80 default_server;
        root /var/www/granz.channel.i03.com;

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
}
```

Then, make a symbolic link to the file in `/etc/nginx/sites-enabled/jarkom`
using `ln -s`. The site should be up and running now. Do the same steps on the remaining workers

<img width="436" alt="Screenshot 2023-11-18 at 21 23 30" src="https://user-images.githubusercontent.com/59758342/284004236-7a5c1c83-13d7-4eaf-bb3e-cc0dad67c4e3.png">

## No. 7

To test the Eisen load balancer, we can use the benchmarking utilities provided by Apache. First, install the `apache2-utils` package

```sh
apt install apache2-utils
```

Then, to make 1000 request with 100 request/second, use the following command

```sh
ab -n 1000 -c 100 "http://10.60.2.2/"
```

## No. 8

We need to perform benchmarking using the various load balancing algorithm provided by `nginx`. First, we need to add all the algorithms into the nginx configuration in the load balancer

File: `/etc/nginx/sites-available/loadbalancer`
```conf
upstream myweb {
	# Round robin (default)
	server 10.60.3.1; # IP Lawine
	server 10.60.3.2; # IP Linie
	server 10.60.3.3; # IP Lugner

	# Weighted round robin
	#server 10.60.3.1 weight=5; # IP Lawine
	#server 10.60.3.2 weight=3; # IP Linie
	#server 10.60.3.3 weight=1; # IP Lugner

	# Least Connection
	#least_conn;
	#server 10.60.3.1; # IP Lawine
	#server 10.60.3.2; # IP Linie
	#server 10.60.3.3; # IP Lugner

	# IP Hash
	#ip_hash;
	#server 10.60.3.1; # IP Lawine
	#server 10.60.3.2; # IP Linie
	#server 10.60.3.3; # IP Lugner

	# Generic Hash
	#hash $request_uri consistent;
	#server 10.60.3.1; # IP Lawine
	#server 10.60.3.2; # IP Linie
	#server 10.60.3.3; # IP Lugner
}

server {
	listen 80 default_server;
	server_name _;

	location / {
		proxy_pass http://myweb;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $http_host;
	}
}
```

Then, using the command `ab`, test the load balancer as follows

```sh
ab -n 200 -c 10 "http://10.60.2.2/"
```

Then, capture the CPU spike usage of each worker using `htop` and the result of the benchmark, then add it to the grimoire

Link to our group's [Grimoire](https://docs.google.com/document/d/1hWokptDIdO2z3M382Ec2-UeIwtB8ROInYjx4lt5w-A0/edit)

## No. 9

To perform the benchmarking in this problem, we first need to set the load balancer configuration to use round robin. Then, we can execute the benchmarking with the following command.

```sh
ab -n 100 -c 10 http://10.60.2.2/
```

We first try with all the worker nodes running, then only two nodes, and finally one. Capture all the benchmark results and visualize it in a graph in the Grimoire

Link to our group's [Grimoire](https://docs.google.com/document/d/1hWokptDIdO2z3M382Ec2-UeIwtB8ROInYjx4lt5w-A0/edit)

## No. 10

We need to add authentication to our load balancer so that it can only be accessed if we can provide the correct credentials. We will need to set the credentials with `htpasswd`. If you already installed the `apache2-utils` package on your machine, you should have the command available

```sh
htpasswd -c .htpasswd netics
```

Then, enter your password for the user `netics`. In our case it would be `ajki03`. Then, move the password file `.htpasswd` to `/etc/nginx/rahasiakita/` directory

```sh
mkdir -p /etc/nginx/rahasiakita
mv .htpasswd /etc/nginx/rahasiakita/.htpasswd
```

Then, we need to add these lines to our nginx load balancer configuration

```conf
location / {
        auth_basic "Restricted Content";
        auth_basic_user_file /etc/nginx/rahasiakita/.htpasswd;
}
```

It should work successfully. If we didn't provide the correct credentials when opening the website using `lynx`, we will not get access into the website. Here's how to get into the website using our credentials

```sh
lynx -auth=netics:ajki03 http://10.60.2.2/
```

## No. 11
## No. 12