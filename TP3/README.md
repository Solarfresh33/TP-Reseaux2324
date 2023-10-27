# TP3 : On va router des trucs
## I - ARP

### 1 - Echange ARP

🌞 **Générer des requêtes ARP**

```
[solar@localhost ~]$ ping 10.3.1.12
PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.959 ms
```
Le ping fonctionne.
```
[solar@localhost ~]$ ip neigh show
10.3.1.12 dev enp0s3 lladdr 08:00:27:b5:20:1b STALE
```
On voit l'adresse MAC de Marcel dans la table ARP de John.
```
[solar@localhost ~]$ ip neigh show

```
[solar@localhost ~]$ ip addr
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b5:20:1b
```
Cette adresse MAC correspond à l'adresse MAC de Marcel.
____

```
[solar@localhost ~]$ sudo tcpdump not port 22 -w partie1TP3-arp.pcap
```
On lance la capture de trames sur John et on la stocke dans un fichier .pcap.
```

### 2. Analyse de trames
🌞 **Analyse de trames**

```
sudo ip neigh flush all
```
On lance la capture avec :
```
sudo tcpdump -i enp0s3 -c 10 -w tp3-arp.pcap not port 22
```
et on ping !

on verifie que le fichier et bien la avec ls puis on se déconnecte aevc exit.

Pour déplacer son fichier on utilise :
```
scp solar@10.3.1.11:/home/solar/tp3-arp.pcap ./
```

## II. Routage

JOHN :
```
[solar@localhost network-scripts]$ cat ifcfg-enp0s3
DEVICE=enp0s3

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.1.11
NETMASK=255.255.255.0
```

MARCEL :
```
[solar@localhost network-scripts]$ cat ifcfg-enp0s3
DEVICE=enp0s3

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.2.12
NETMASK=255.255.255.0
```

ROUTEUR :
```
[solar@localhost network-scripts]$ cat ifcfg-enp0s3
DEVICE=enp0s3

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.1.254
NETMASK=255.255.255.0
```

```
[solar@localhost network-scripts]$ cat ifcfg-enp0s8
DEVICE=enp0s3

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.3.2.254
NETMASK=255.255.255.0
```

---

Test ping :
```
--- 10.3.1.254 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4009ms
rtt min/avg/max/mdev = 1.783/2.073/2.499/0.237 ms
```

```
--- 10.3.2.254 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3010ms
rtt min/avg/max/mdev = 1.514/1.835/2.215/0.317 ms
```

### 1. Mise en place du routage

🌞 **Ajouter les routes statiques nécessaires pour que john et marcel puissent se ping**

Pour John :
```
sudo ip route add 10.3.2.0/24 via 10.3.1.254 dev enp0s3
---
[solar@localhost network-scripts]$ ip r s
10.3.1.0/24 dev enp0s3 proto kernel scope link src 10.3.1.11 metric 100
10.3.2.0/24 via 10.3.1.254 dev enp0s3
```
Pour Marcel :
```
sudo ip route add 10.3.1.0/24 via 10.3.2.254 dev enp0s3
 ---
[solar@localhost network-scripts]$ ip route
 show
10.3.1.0/24 via 10.3.2.254 dev enp0s3
10.3.2.0/24 dev enp0s3 proto kernel scope link src 10.3.2.12 metric 100
```
### 2. Analyse de trames

🌞 **Analyse des échanges ARP**

```
sudo ip neigh flush all
```
| ordre | type trame  | IP source | MAC source                | IP destination | MAC destination            |
| ----- | ----------- | --------- | ------------------------- | -------------- | -------------------------- |
| 1     | Requête ARP | x         |`marcel` `08:00:27:9e:1d:39`| x             | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | x         |`john` `08:00:27:89:02:33` | x              |`marcel` `08:00:27:9e:1d:39`|
| ...   | ...         | ...       |...                        |                |                            |
| 3     | Ping        | 10.3.2.12 |`marcel` `08:00:27:9e:1d:39`| 10.3.1.11     |`john` `08:00:27:89:02:33`  |
| 4     | Pong        | 10.3.1.11 |`john` `08:00:27:89:02:33` | 10.3.2.12      |`marcel` `08:00:27:9e:1d:39`|

### 3. Accès internet
🌞 **Donnez un accès internet à vos machines - config routeur**

Vérification du NAT:
