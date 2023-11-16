# TP5 : TCP, UDP et services réseau

## I. First steps

🌞 **Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP**

- `Spotify`
  - Local : `41234`
  - Distant : `35.186.224.47:443`
- `opera`
    - Local : `49054`
    - Distant : `35.186.224.47:443`
- `Discord`
  - Local : `37364`
  - Distant : `162.159.128.233:443`
- `Epic Games Launcher`
  - Local : `37524`
  - Distant : `23.23.106.155:443`
- `VSCode`
  - Local : `37348`
  - Distant : `51.144.164.215:443`

<br>
🌞 **Demandez l'avis à votre OS**

```
bash
[solar@ROG14Max TP5]$ sudo ss -ptun
Netid            State                 Recv-Q            Send-Q                             Local Address:Port                               Peer Address:Port             Process                                                 
udp              ESTAB                 0                 0                                   10.33.16.229:46366                             35.186.224.25:443               users:(("Discord",pid=26377,fd=27))                    
udp              ESTAB                 0                 0                                   10.33.16.229:49054                           162.159.128.233:443               users:(("Discord",pid=26377,fd=25))                    
udp              ESTAB                 0                 0                            10.33.16.229%wlp4s0:68                                 10.33.19.254:67                users:(("NetworkManager",pid=1298,fd=26))              
tcp              ESTAB                 0                 0                                   10.33.16.229:40528                              66.102.1.188:5228              users:(("opera",pid=22164,fd=75))                      
tcp              ESTAB                 0                 0                                   10.33.16.229:52748                           162.159.133.234:443               users:(("Discord",pid=26377,fd=145))                   
tcp              ESTAB                 0                 0                                      127.0.0.1:54112                                 127.0.0.1:63342             users:(("jcef_helper",pid=24394,fd=26))                
toperacp              ESTAB                 0                 0                                   10.33.16.229:52498                              51.195.5.156:443               users:(("anydesk",pid=1526,fd=45))                     
tcp              ESTAB                 0                 0                                   10.33.16.229:42698                            185.26.182.111:443               users:(("opera",pid=22164,fd=25))                      
tcp              ESTAB                 0                 0                                   10.33.16.229:45692                             35.186.224.25:443               users:(("Discord",pid=26377,fd=28))                    
tcp              ESTAB                 0                 0                                   10.33.16.229:58388                             20.250.85.194:443               users:(("copilot-agent-l",pid=24357,fd=20))            
tcp              CLOSE-WAIT            25                0                                   10.33.16.229:43750                             82.145.216.16:443               users:(("opera",pid=22164,fd=32))                      
tcp              ESTAB                 0                 0                                   10.33.16.229:53736                           192.241.190.146:443               users:(("opera",pid=22164,fd=69))                      
tcp              ESTAB                 0                 0                                   10.33.16.229:33522                             140.82.112.26:443               users:(("opera",pid=22164,fd=53))                      
tcp              ESTAB                 0                 585                                 10.33.16.229:33544                              3.213.18.157:443               users:(("opera",pid=22164,fd=29))                      
tcp              ESTAB                 0                 0                             [::ffff:127.0.0.1]:63342                        [::ffff:127.0.0.1]:54112             users:(("java",pid=24205,fd=35))  
```

# II. Mise en place

## 1. SSH

🌞 **Examinez le trafic dans Wireshark**

[Voir fichier Capture_SSH](Capture_SSH.pcapng)

🌞 **Demandez aux OS**

```
bash
solar@ROG14Max ~ $ ss -tpun
Netid   State        Recv-Q   Send-Q           Local 
Address:Port             Peer Address:Port    Process                                   
udp     ESTAB        0        0          10.33.16.229%wlp4s0:68               10.33.19.254:67                                                
tcp     ESTAB        0        0                    127.0.0.1:40206               127.0.0.1:63342    users:(("jcef_helper",pid=4985,fd=28))   
tcp     ESTAB        0        0                 10.33.16.229:38938           136.243.137.2:443                                               
tcp     CLOSE-WAIT   25       0                 10.33.16.229:48346           82.145.216.20:443      users:(("opera",pid=3614,fd=29))         
tcp     ESTAB        0        0                 10.33.16.229:60326          74.125.206.188:5228     users:(("opera",pid=3614,fd=25))         
tcp     ESTAB        0        0                 10.33.16.229:60830         192.241.190.146:443      users:(("opera",pid=3614,fd=55))         
tcp     ESTAB        0        0                     10.4.1.1:60002               10.4.1.11:22       users:(("ssh",pid=7499,fd=3))            
tcp     ESTAB        0        0                    127.0.0.1:56418               127.0.0.1:63342    users:(("jcef_helper",pid=4985,fd=25))   
tcp     ESTAB        0        0                 10.33.16.229:35410           140.82.114.26:443      users:(("opera",pid=3614,fd=83))         
tcp     ESTAB        0        0           [::ffff:127.0.0.1]:63342      [::ffff:127.0.0.1]:56418    users:(("java",pid=4792,fd=52))          
tcp     ESTAB        0        0           [::ffff:127.0.0.1]:63342      [::ffff:127.0.0.1]:40206    users:(("java",pid=4792,fd=329)) 
```

🌞Examinez le trafic dans Wireshark

*tp5_3way_handshake.pcapng*

- __déterminez si SSH utilise TCP ou UDP__

> Transmission Control Protocol, Src Port: 58078, Dst Port: 22, Seq: 0, Len: 0

- __repérez le 3-Way Handshake à l'établissement de la connexion__

> Les 3 premières frames avec les flags SYN/SYNACK/ACK

- __repérez du trafic SSH__

> Les frames avec le protocol SSHv2

- __repérez le FIN ACK à la fin d'une connexion__

> les deux avants dernières frames avec le flag FIN ACK

---
🌞 Demandez aux OS

De ma machine :
```
  Proto  Adresse locale         Adresse distante       État
  TCP    10.5.1.1:59841         10.5.1.11:ssh          ESTABLISHED
```

De la vm :
```
tcp      ESTAB    0          0                                 10.5.1.11:ssh               10.5.1.1:59841
```

### 2. Routage
---
🌞 Prouvez que

```
[solar@node1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=21.7 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=18.5 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=19.3 ms
^C
--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 18.519/19.824/21.703/1.361 ms
```

```
[solar@node1 ~]$ ping ynov.com
PING ynov.com (104.26.10.233) 56(84) bytes of data.
64 bytes from 104.26.10.233 (104.26.10.233): icmp_seq=1 ttl=55 time=21.4 ms
64 bytes from 104.26.10.233 (104.26.10.233): icmp_seq=2 ttl=55 time=23.1 ms
64 bytes from 104.26.10.233 (104.26.10.233): icmp_seq=3 ttl=55 time=22.8 ms
^C64 bytes from 104.26.10.233: icmp_seq=4 ttl=55 time=22.1 ms

--- ynov.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 15180ms
rtt min/avg/max/mdev = 21.379/22.350/23.107/0.664 ms
```

### 3. Serveur Web
---
🌞 Installez le paquet nginx

```
sudo dnf install nginx
```

🌞 Créer le site web

```
[solar@web ~]$ cd /var/
[solar@web var]$ sudo mkdir www
[solar@web var]$ cd www
[solar@web www]$ sudo mkdir site_web_nul
[solar@web www]$ cd site_web_nul/
[solar@web site_web_nul]$ sudo touch index.html
[solar@web site_web_nul]$ sudo nano index.html
[solar@web site_web_nul]$ sudo cat index.html
<h1>MEOW <3</h1>
```

🌞 Donner les bonnes permissions

> sudo chown -R nginx:nginx /var/www/site_web_nul

🌞 Créer un fichier de configuration NGINX pour notre site web

> cd /etc/nginx/conf.d/
```
sudo touch site_web_nul.conf

sudo nano site_web_nul.conf
```

```
[solar@web conf.d]$ sudo cat site_web_nul.conf
server {
  # le port sur lequel on veut écouter
  listen 80;

  # le nom de la page d'accueil si le client de la précise pas
  index index.html;

  # un nom pour notre serveur (pas vraiment utile ici, mais bonne pratique)
  server_name www.site_web_nul.b1;

  # le dossier qui contient notre site web
  root /var/www/site_web_nul;
}
```

🌞 Démarrer le serveur web !

> sudo systemctl start nginx

> systemctl status nginx 
```
[solar@web ~]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Tue 2023-11-14 11:42:02 CET; 31s ago
    Process: 1376 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1377 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1378 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1379 (nginx)
      Tasks: 2 (limit: 4674)
     Memory: 3.2M
        CPU: 61ms
     CGroup: /system.slice/nginx.service
             ├─1379 "nginx: master process /usr/sbin/nginx"
             └─1380 "nginx: worker process"

Nov 14 11:42:02 web systemd[1]: Starting The nginx HTTP and reverse proxy server...
Nov 14 11:42:02 web nginx[1377]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Nov 14 11:42:02 web nginx[1377]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Nov 14 11:42:02 web systemd[1]: Started The nginx HTTP and reverse proxy server.
```

🌞 Ouvrir le port firewall

```
[solar@web ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
[solar@web ~]$ sudo firewall-cmd --reload
success
```

🌞 Visitez le serveur web !

> http://10.5.1.12

```
[solar@node1 ~]$ curl http://10.5.1.12
<h1>MEOW <3</h1>
```

🌞 Visualiser le port en écoute

```
ss -ln | grep ":80"
```

```
tcp   LISTEN 0      511                                       0.0.0.0:80               0.0.0.0:*
tcp   LISTEN 0      511                                          [::]:80                  [::]:*
```

🌞 Analyse trafic

*tp5_web.pcapng*
