# TP2 : Ethernet, IP, et ARP
## I. Setup IP

🌞 Mettez en place une configuration réseau fonctionnelle entre les deux machines
```
interface ip set address name="Ethernet" static 192.168.30.2 255.255.255.252 192.168.30.1
```
```
PS C:\Users\maxim> ipconfig /all

Carte Ethernet Ethernet 2 :

   Adresse physique . . . . . . . . . . . : 00-FF-A4-7E-E1-6B
   DHCP activé. . . . . . . . . . . . . . : Non
   Configuration automatique activée. . . : Oui
   Adresse IPv4. . . . . . . . . . . . . .: 192.168.30.2(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.255.252
   Passerelle par défaut. . . . . . . . . : 192.168.30.1
```
🌞 Prouvez que la connexion est fonctionnelle entre les deux machines
```
PS C:\Users\maxim> ping 192.168.30.1

Envoi d’une requête 'Ping'  192.168.30.1 avec 32 octets de données :
Réponse de 192.168.30.1 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.30.1 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.30.1 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.30.1 : octets=32 temps=2 ms TTL=128

Statistiques Ping pour 192.168.30.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 2ms, Maximum = 2ms, Moyenne = 2ms
```
🌞 Wireshark it
Les deux types utiliser sont le type 8 qui correspond à Echo Request et le type 0 qui correspond à Echo reply (pingpong)

___
___

## II. ARP my bro

🌞 Check the ARP table

MAC du Binome:
```
PS C:\WINDOWS\system32> arp -a

f0-2f-74-4d-0c-32
```

MAC Gatewway:
```
PS C:\WINDOWS\system32> arp -a

10.33.19.254          00-c0-e7-e0-04-4e     dynamique
```

🌞 Manipuler la table ARP

Pour vider sa table ARP il faut faire:
```
arp -d *
```

```
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.30.2 --- 0xa
  Adresse Internet      Adresse physique      Type
  192.168.30.1          f0-2f-74-4d-0c-32     dynamique
  224.0.0.2             01-00-5e-00-00-02     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

PS C:\WINDOWS\system32> arp -d *
PS C:\WINDOWS\system32> arp /a

Interface : 192.168.30.2 --- 0xa
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

PS C:\WINDOWS\system32> ping 192.168.30.1

Envoi d’une requête 'Ping'  192.168.30.1 avec 32 octets de données :
Réponse de 192.168.30.1 : octets=32 temps=4 ms TTL=128
Réponse de 192.168.30.1 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.30.1 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.30.1 : octets=32 temps=2 ms TTL=128

Statistiques Ping pour 192.168.30.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 2ms, Maximum = 4ms, Moyenne = 2ms
    
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.30.2 --- 0xa
  Adresse Internet      Adresse physique      Type
  192.168.30.1          f0-2f-74-4d-0c-32     dynamique
  224.0.0.2             01-00-5e-00-00-02     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
```

🌞 Wireshark it

 Adresse expediteur (adresse mac de ma machine): ASUSTekC_4d:0c:32 (f0:2f:74:4d:0c:32) 
 Adresse destination (adresse mac de sa machine): ASUSTekC_4d:0c:32 (f0:2f:74:4d:0c:32)

---

---

## III. DHCP you too my brooo

🌞 Wireshark it

Tram 1: Discovert
Source: 0.0.0.0
Destination: 255.255.255.255

Tram 2: Offer
Source: 10.33.19.254
Destination:10.33.19.80

Tram 3: Request
Source: 0.0.0.0
Destination: 255.255.255.255