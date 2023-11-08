# TP4 : DHCP
## I : DHCP Client

🌞 **Déterminer**

- l'adresse du serveur DHCP
- l'heure exacte à laquelle vous avez obtenu votre bail DHCP
- l'heure exacte à laquelle il va expirer

```	
PS C:\Users\maxim> ipconfig /all

 Serveur DHCP . . . . . . . . . . . . . : 10.33.79.254
Bail obtenu. . . . . . . . . . . . . . : vendredi 27 octobre 2023 09:08:05
Bail expirant. . . . . . . . . . . . . : samedi 28 octobre 2023 09:07:56
```

 🌞 **Capturer un échange DHCP**

- forcer votre OS à refaire un échange DHCP complet
- utiliser Wireshark pour capturer les 4 trames DHCP

```	
PS C:\Users\maxim> ipconfig /release
```
Le bail est libéré. La requête DHCP est restart automatiquement.

🌞 Analyser la capture Wireshark

- Parmi ces 4 trames, laquelle contient les informations proposées au client ?
- En cliquant sur l'une des 4 trames, et en dépliant la partie DHCP (en bas dans l'interface de Wireshark) vous pourrez repérer ces informations

```
Les trames DHCP OFFER et DHCP ACK contiennent les informations proposées au client. Elles sont envoyé par le serveur DHCP, afin de proposer une adresse IP au client.
```

## II : Serveur DHCP 

### 3. Setup topologie

🌞 Preuve de mise en place

```
sudo ip route add default via 10.4.1.254 dev enp0s3
```

__DHCP :__
```
[solar@dhcp ~]$ ping google.com
PING google.com (172.217.19.142) 56(84) bytes of data.
64 bytes from mrs08s04-in-f14.1e100.net (172.217.19.142): icmp_seq=1 ttl=110 time=26.0 ms
64 bytes from mrs08s04-in-f14.1e100.net (172.217.19.142): icmp_seq=2 ttl=110 time=27.0 ms
64 bytes from mrs08s04-in-f14.1e100.net (172.217.19.142): icmp_seq=3 ttl=110 time=28.1 ms
^C64 bytes from 172.217.19.142: icmp_seq=4 ttl=110 time=25.2 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 9384ms
rtt min/avg/max/mdev = 25.247/26.599/28.115/1.077 ms
```

__Node2 :__
```
[solar@node2 ~]$ ping google.com
PING google.com (172.217.19.142) 56(84) bytes of data.
64 bytes from mrs08s04-in-f14.1e100.net (172.217.19.142): icmp_seq=1 ttl=110 time=29.6 ms
64 bytes from mrs08s04-in-f14.1e100.net (172.217.19.142): icmp_seq=2 ttl=110 time=43.5 ms
^C64 bytes from 172.217.19.142: icmp_seq=3 ttl=110 time=33.9 ms

--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 6371ms
rtt min/avg/max/mdev = 29.615/35.685/43.513/5.808 ms
```     

__Traceroute :__
```
[solar@node2 ~]$ traceroute google.com
traceroute to google.com (172.217.19.142), 30 hops max, 60 byte packets
 1  _gateway (10.4.1.254)  52.251 ms  47.549 ms  2.101 ms
 2  10.0.3.2 (10.0.3.2)  9.986 ms  8.212 ms  5.919 ms
```
---
### 4. Serveur DHCP

🌞 Rendu

> dnf -y install dhcp-server
>
>sudo nano /etc/dhcp/dhcpd.conf
>
>sudo cat /etc/dhcp/dhcpd.conf
```
# create new
# specify domain name
option domain-name     "srv.world";
# specify DNS server's hostname or IP address
option domain-name-servers     dlp.srv.world;
# default lease time
default-lease-time 600;
# max lease time
max-lease-time 7200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.4.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.4.1.137 10.4.1.237;
    # specify broadcast address
    option broadcast-address 10.4.1.255;
    # specify gateway
    option routers 10.4.1.1;
}
```
>sudo systemctl enable --now dhcpd
```
[solar@dhcp ~]$ sudo systemctl enable --now dhcpd
Created symlink /etc/systemd/system/multi-user.target.wants/dhcpd.service → /usr/lib/systemd/system/dhcpd.service.
```

Status :
```
[solar@dhcp ~]$ systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
     Active: active (running) since Tue 2023-11-07 16:55:45 CET; 3min 43s ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 11349 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 4674)
     Memory: 5.3M
        CPU: 53ms
     CGroup: /system.slice/dhcpd.service
             └─11349 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid
```
---

### 5. Client DHCP

🌞 Test !

>sudo touch ifcfg-enp0s3
>
>sudo nano ifcfg-enp0s3
>
```
DEVICE=enp0s8

BOOTPROTO=dhcp
ONBOOT=yes
```
```
sudo nmcli con reload
sudo nmcli con up "System enp0s3"
sudo systemctl restart NetworkManager

```

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4b:cf:e3 brd ff:ff:ff:ff:ff:ff
    inet 10.4.1.137/24 brd 10.4.1.255 scope global dynamic noprefixroute enp0s3
```
---
🌞 Prouvez que
```
Nov 08 18:00:30 dhcp dhcpd[759]: DHCPDISCOVER from 08:00:27:4b:cf:e3 via enp0s3
Nov 08 18:00:34 dhcp dhcpd[759]: DHCPOFFER on 10.4.1.137 to 08:00:27:4b:cf:e3 (node1) via enp0s3
Nov 08 18:00:34 dhcp dhcpd[759]: DHCPDISCOVER from 08:00:27:4b:cf:e3 (node1) via enp0s3
Nov 08 18:00:34 dhcp dhcpd[759]: DHCPOFFER on 10.4.1.137 to 08:00:27:4b:cf:e3 (node1) via enp0s3
Nov 08 18:00:34 dhcp dhcpd[759]: DHCPREQUEST for 10.4.1.137 (10.4.1.253) from 08:00:27:4b:cf:e3 (node1) via enp0s3
Nov 08 18:00:34 dhcp dhcpd[759]: DHCPACK on 10.4.1.137 to 08:00:27:4b:cf:e3 (node1) via enp0s3
```


---
🌞 Bail DHCP serveur

>cat /var/lib/dhcpd/dhcpd.leases

```
lease 10.4.1.137 {
  starts 3 2023/11/08 17:45:05;
  ends 3 2023/11/08 17:55:05;
  cltt 3 2023/11/08 17:45:05;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:4b:cf:e3;
  uid "\001\010\000'K\317\343";
  client-hostname "node1";
}
```
---

### 6. Option DHCP

🌞 Nouvelle conf !
```
[solar@dhcp ~]$ sudo cat /etc/dhcp/dhcpd.conf
# create new
# specify domain name
option domain-name     "srv.world";
# specify DNS server's hostname or IP address
option domain-name-servers     8.8.8.8;
# default lease time
default-lease-time 21600;
# max lease time
max-lease-time 22000;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.4.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.4.1.137 10.4.1.237;
    # specify broadcast address
    option broadcast-address 10.4.1.255;
    # specify gateway
    option routers 10.4.1.254;
}
```
> sudo systemctl restart dhcpd

🌞 Test !

__Dans node1 :__
>sudo dhclient -r enp0s3
>sudo dhclient enp0s3

__Serveur DNS :__
```
[solar@node1 ~]$ cat /etc/resolv.conf
# Generated by NetworkManager
search srv.world
google.com 8.8.8.8
```

__Nouvelle route :__

```
[solar@node1 ~]$ ip r s
default via 10.4.1.254 dev enp0s3
efault via 10.4.1.254 dev enp0s3 proto dhcp src 10.4.1.137 metric 100
10.4.1.0/24 dev enp0s3 proto kernel scope link src 10.4.1.137 metric 100
```

__Durée bail DHCP :__
```
[solar@node1 ~]$ cat /var/lib/dhclient/dhclient.leases
lease {
  interface "enp0s3";
  fixed-address 10.4.1.138;
  option subnet-mask 255.255.255.0;
  option routers 10.4.1.254;
  option dhcp-lease-time 21600;
  option dhcp-message-type 5;
  option domain-name-servers 8.8.8.8;
  option dhcp-server-identifier 10.4.1.253;
  option broadcast-address 10.4.1.255;
  option domain-name "srv.world";
  renew 3 2023/11/08 20:29:56;
  rebind 3 2023/11/08 23:20:00;
  expire 4 2023/11/09 00:05:00;
}
```

__Preuve internet :__

```
[solar@node1 ~]$ ping google.com
PING google.com (172.217.18.206) 56(84) bytes of data.
64 bytes from ham02s14-in-f206.1e100.net (172.217.18.206): icmp_seq=1 ttl=115 time=19.8 ms
64 bytes from par10s38-in-f14.1e100.net (172.217.18.206): icmp_seq=2 ttl=115 time=19.3 ms
64 bytes from ham02s14-in-f206.1e100.net (172.217.18.206): icmp_seq=3 ttl=115 time=19.9 ms
^C64 bytes from 172.217.18.206: icmp_seq=4 ttl=115 time=20.0 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 9377ms
rtt min/avg/max/mdev = 19.321/19.743/19.952/0.256 ms
```
🌞 Capture Wireshark
