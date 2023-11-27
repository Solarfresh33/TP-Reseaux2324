🌞 Dans le rendu, je veux

- un cat des 3 fichiers de conf (fichier principal + deux fichiers de zone)
- un systemctl status named qui prouve que le service tourne bien
- une commande ss qui prouve que le service écoute bien sur un port

```
$ sudo cat /etc/named.conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
        listen-on port 53 { 127.0.0.1;any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost;any; };
        allow-query-cache {localhost; any; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "tp6.b1" IN {
        type master;
        file "tp6.b1.db";
        allow-update { none; };
        allow-query { any; };
};

zone "1.4.10.in-addr.arpa" IN {
     type master;
     file "tp6.b1.rev";
     allow-update { none; };
     allow-query { any; };

};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```
```	
$ sudo cat /var/named/tp6.b1.db
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns       IN A 10.6.1.101
john      IN A 10.6.1.11
```
```
$ sudo cat /var/named/tp6.b1.rev
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Reverse lookup
101 IN PTR dns.tp6.b1.
11 IN PTR john.tp6.b1.
```
```
sudo systemctl status named
[sudo] password for solar:
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; preset: disabled)
     Active: active (running) since Fri 2023-11-24 10:20:16 CET; 59min ago
   Main PID: 11576 (named)
      Tasks: 5 (limit: 4674)
     Memory: 18.1M
        CPU: 69ms
     CGroup: /system.slice/named.service
             └─11576 /usr/sbin/named -u named -c /etc/named.conf

Nov 24 10:20:16 localhost.localdomain named[11576]: zone 1.0.0.127.in-addr.arpa/IN: loaded serial 0
Nov 24 10:20:16 localhost.localdomain named[11576]: zone tp6.b1/IN: loaded serial 2019061800
Nov 24 10:20:16 localhost.localdomain named[11576]: zone localhost/IN: loaded serial 0
Nov 24 10:20:16 localhost.localdomain named[11576]: zone 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip>
Nov 24 10:20:16 localhost.localdomain named[11576]: zone localhost.localdomain/IN: loaded serial 0
Nov 24 10:20:16 localhost.localdomain named[11576]: all zones loaded
Nov 24 10:20:16 localhost.localdomain systemd[1]: Started Berkeley Internet Name Domain (DNS).
Nov 24 10:20:16 localhost.localdomain named[11576]: running
Nov 24 10:20:16 localhost.localdomain named[11576]: managed-keys-zone: Initializing automatic trust anchor management for z>
Nov 24 10:20:16 localhost.localdomain named[11576]: resolver priming query complete
lines 1-20/20 (END)
```
```	
$ ss -atnl
State         Recv-Q        Send-Q               Local Address:Port                 Peer Address:Port        Process
LISTEN        0             10                       127.0.0.1:53                        0.0.0.0:*
LISTEN        0             128                        0.0.0.0:22                        0.0.0.0:*
LISTEN        0             10                      10.6.1.101:53                        0.0.0.0:*
LISTEN        0             4096                     127.0.0.1:953                       0.0.0.0:*
LISTEN        0             10                           [::1]:53                           [::]:*
LISTEN        0             128                           [::]:22                           [::]:*
LISTEN        0             4096                         [::1]:953                          [::]:*
```

🌞 Ouvrez le bon port dans le firewall
    
 ```
 $ sudo firewall-cmd --add-port=53/tcp --permanent
success
$ sudo firewall-cmd --reload
success
```
🌞 Sur la machine john.tp6.b1

```
[solar@john ~]$ sudo cat /etc/resolv.conf
[sudo] password for solar:
# Generated by NetworkManager
nameserver 10.6.1.101
```

```
[solar@john ~]$ sudo cat  /etc/sysconfig/network-scripts/ifcfg-enp0s3
DEVICE=enp0s3

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.6.1.11
NETMASK=255.255.255.0

DNS1=10.6.1.101
```

```

PING :
[solar@john ~]$ ping dns.tp6.b1
PING dns.tp6.b1 (10.6.1.101) 56(84) bytes of data.
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=1 ttl=64 time=2.45 ms
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=2 ttl=64 time=1.30 ms
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=3 ttl=64 time=1.77 ms
^C
--- dns.tp6.b1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms

_________________________________

[solar@john ~]$ ping ynov.com
PING ynov.com (172.67.74.226) 56(84) bytes of data.
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=1 ttl=56 time=22.1 ms
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=2 ttl=56 time=22.2 ms
64 bytes from 172.67.74.226 (172.67.74.226): icmp_seq=3 ttl=56 time=37.6 ms
^C
--- ynov.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2248ms
```
---
🌞 Sur votre PC

> sudo tcpdump -i enp0s3 -c 10 -w tp6-dns.pcap not port 22

__-> voir tp6-dns.pcap__

## III. Serveur DHCP

🌞 Installer un serveur DHCP

> sudo cat /etc/dhcp/dhcpd.conf

```
[solar@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
option domain-name     "srv.world";
# specify DNS server's hostname or IP address
option domain-name-servers     10.6.1.101;
# default lease time
default-lease-time 600;
# max lease time
max-lease-time 7200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.6.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.6.1.13 10.6.1.37;
    # specify broadcast address
    option broadcast-address 10.6.1.255;
    # specify gateway
    option routers 10.6.1.254;
}
```
---
🌞 Test avec john.tp6.b1

1. Vérifiez l'adresse IP dans la bonne plage :
```
[solar@john ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:13:3a:cd brd ff:ff:ff:ff:ff:ff
    inet 10.6.1.13/24 brd 10.6.1.255 scope global dynamic noprefixroute enp0s3
       valid_lft 536sec preferred_lft 536sec
    inet6 fe80::a00:27ff:fe13:3acd/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```
---
2. Vérifiez la passerelle (routeur) automatiquement configurée :
```
[solar@john ~]$ ip r
default via 10.6.1.254 dev enp0s3 proto dhcp src 10.6.1.13 metric 100
10.6.1.0/24 dev enp0s3 proto kernel scope link src 10.6.1.13 metric 100
```
---
3. Vérifiez le serveur DNS automatiquement configuré :
```
[solar@john ~]$ sudo cat /etc/resolv.conf
[sudo] password for solar:
# Generated by NetworkManager
search srv.world
nameserver 10.6.1.101
```
---
4. Vérifiez l'accès à Internet :
```
[solar@john ~]$ ping google.com
PING google.com (142.250.75.238) 56(84) bytes of data.
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=1 ttl=115 time=12.7 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=2 ttl=115 time=13.9 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 12.725/13.306/13.887/0.581 ms
```
---
🌞 Requête web avec john.tp6.b1

> sudo tcpdump -i enp0s3 -c 50 -w tp6-web.pcap not port 22

---> voir tp6-web.pcap
