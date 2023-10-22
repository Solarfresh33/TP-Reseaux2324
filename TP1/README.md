# TP 1 Fonctionnement réseaux


🌞 **Affichez les infos des cartes réseau de votre PC**

```
PS C:\Users\maxim> ipconfig /all

Carte réseau sans fil Wi-Fi :

Suffixe DNS propre à la connexion. . . :
 Description. . . . . . . . . . . . . . : MediaTek Wi-Fi 6E MT7922 (RZ616) 160MHz Wireless LAN Card
   Adresse physique . . . . . . . . . . . : BC-F4-D4-E1-C1-5B
   DHCP activé. . . . . . . . . . . . . . : Oui
   Configuration automatique activée. . . : Oui
   Adresse IPv6 de liaison locale. . . . .: fe80::c4e6:c919:d4f6:f354%19(préféré)
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.48.119(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.252.0
   Bail obtenu. . . . . . . . . . . . . . : mercredi 11 octobre 2023 09:04:40
   Bail expirant. . . . . . . . . . . . . : jeudi 12 octobre 2023 09:04:40
   Passerelle par défaut. . . . . . . . . : 10.33.51.254
   Serveur DHCP . . . . . . . . . . . . . : 10.33.51.254
   IAID DHCPv6 . . . . . . . . . . . : 163378388
   DUID de client DHCPv6. . . . . . . . : 00-01-00-01-2B-C5-AE-52-3C-18-A0-C9-73-4A
   Serveurs DNS. . .  . . . . . . . . . . : 10.33.10.2
                                       8.8.8.8
   NetBIOS sur Tcpip. . . . . . . . . . . : Activé
 ```
_______
 🌞 **Trouvez comment afficher les informations sur une carte IP (change selon l'OS)**

 ```
Carte réseau sans fil Wi-Fi :
   Adresse physique (MAC). . . . . . . . . . . : BC-F4-D4-E1-C1-5B
   Adresse IPv6 de liaison locale. . . . .: fe80::c4e6:c919:d4f6:f354%19(préféré)
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.48.119(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.252.0
   Passerelle par défaut. . . . . . . . . : 10.33.51.254
 ```

____

🌞 **Déterminer la MAC de la passerelle**

```
PS C:\Users\maxim> arp -a

Interface : 10.33.48.119 --- 0x13
  Adresse Internet      Adresse physique      Type
  10.33.48.118          dc-46-28-b5-66-57     dynamique
  10.33.51.254          7c-5a-1c-cb-fd-a4     dynamique
  10.33.51.255          ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
  ```


🌞 **Modifiez l'IP des deux machines pour qu'elles soient dans le même réseau**

Nouvelle IP : 10.10.10.11

Masque : 255.255.255.0

---
🌞 **Vérifier à l'aide d'une commande que votre IP a bien été changée**

```
Avec ipconfig :

Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.11
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
```
Envoi d'un ping :
```
 ping 10.10.10.12

Envoi d’une requête 'Ping'  10.10.10.12 avec 32 octets de données :
Réponse de 10.10.10.12 : octets=32 temps=5 ms TTL=128
Réponse de 10.10.10.12 : octets=32 temps=5 ms TTL=128
Réponse de 10.10.10.12 : octets=32 temps=2 ms TTL=128
Réponse de 10.10.10.12 : octets=32 temps=3 ms TTL=128

Statistiques Ping pour 10.10.10.12:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 2ms, Maximum = 5ms, Moyenne = 3ms
```

---
🌞 **Déterminer l'adresse MAC de votre correspondant**

Avec :
```
arp -a
```
Adresse MAC du correspondant :
```
 Adresse Internet      Adresse physique      Type
  10.10.10.12           08-bf-b8-c2-2a-57     dynamique
```
### 4. Petit chat privé
---
🌞 **sur le PC client avec par exemple l'IP 192.168.1.2**

Client :
```
Cmd line: 10.10.10.12 8888
K
coucou  
```

---
🌞 **Visualiser la connexion en cours :**
```
 [nc64.exe]
  TCP    10.33.48.106:139       0.0.0.0:0              LISTENING
 Impossible d’obtenir les informations de propriétaire
  TCP    10.33.48.106:49413     20.199.120.182:443     ESTABLISHED
  WpnService
```

### 5. Firewall
---
🌞 **Activez et configurez votre pare-feu**

On autorise avec les paramètres avancées, on active les règles sur "oui" pour netcat.


🌞 **Exploration du DHCP, depuis votre PC**

```
ipconfig /all 

Carte réseau sans fil Wi-Fi :

   Bail obtenu. . . . . . . . . . . . . . : lundi 16 octobre 2023 09:02:08
   Bail expirant. . . . . . . . . . . . . : mardi 17 octobre 2023 09:02:08
   Serveur DHCP . . . . . . . . . . . . . : 10.33.51.254
 ```

 🌞 **Trouver l'adresse IP du serveur DNS que connaît votre ordinateur**

 ```
 ipconfig /all

 Carte réseau sans fil Wi-fi :

   Serveurs DNS. . .  . . . . . . . . . . : 10.33.10.2
                                       8.8.8.8
```
______
🌞 **Utiliser, en ligne de commande l'outil nslookup (Windows, MacOS) ou dig (GNU/Linux, MacOS) pour faire des requêtes DNS à la main**

```
nslookup google.com 8.8.8.8
Serveur :   dns.google
Address:  8.8.8.8

Réponse ne faisant pas autorité :
Nom :    google.com
Addresses:  2a00:1450:4007:818::200e
          142.250.179.110


nslookup 231.34.113.12
DNS request timed out.
    timeout was 2 seconds.
Serveur :   UnKnown
Address:  10.33.10.2

DNS request timed out.
    timeout was 2 seconds.
*** Le délai de la requête sur UnKnown est dépassé.
PS C:\Users\maxim> nslookup 78.34.2.17
DNS request timed out.
    timeout was 2 seconds.
Serveur :   UnKnown
Address:  10.33.10.2

DNS request timed out.
    timeout was 2 seconds.
*** Le délai de la requête sur UnKnown est dépassé.
```
_____

