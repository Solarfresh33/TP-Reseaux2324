# TP7 : Do u secure

## II. SSH

### 1. Fingerprint

🌞 Effectuez une connexion SSH en vérifiant le fingerprint

```
PS C:\Users\maxim> ssh solar@10.7.1.11
The authenticity of host '10.7.1.11 (10.7.1.11)' can't be established.
ED25519 key fingerprint is SHA256:9EhlY279YtTMkCFfvAG8Xo7xKdUmdDvXO2XTyqPr2Bk.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.7.1.11' (ED25519) to the list of known hosts.
solar@10.7.1.11's password:
```

>  sudo ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key
```
[solar@john ~]$ sudo ssh-keygen -l -f /etc/ssh/ssh_host_ed25519_key
[sudo] password for solar:
256 SHA256:9EhlY279YtTMkCFfvAG8Xo7xKdUmdDvXO2XTyqPr2Bk /etc/ssh/ssh_host_ed25519_key.pub (ED25519)
```

## 2. Conf serveur SSH

🌞 Consulter l'état actuel

```
[solar@routeur ~]$ sudo ss -alntpu
tcp     LISTEN   0        128                0.0.0.0:22              0.0.0.0:*       users:(("sshd",pid=685,fd=3))
```
---
🌞 Modifier la configuration du serveur SSH
```
[solar@routeur ~]$ sudo cat /etc/ssh/sshd_config
Port 22000
#AddressFamily any
ListenAddress 10.7.1.254
#ListenAddress ::
```
---
🌞 Prouvez que le changement a pris effet
```
[solar@routeur ~]$ sudo ss -alntpu
tcp     LISTEN   0        128            10.7.1.254:22000            0.0.0.0:*       users:(("sshd",pid=1399,fd=3))
```
---
🌞 N'oubliez pas d'ouvrir ce nouveau port dans le firewall

```
[solar@routeur ~]$ sudo firewall-cmd --add-port=22000/tcp --permanent
[sudo] password for solar:
success
[solar@routeur ~]$ sudo firewall-cmd --reload
success
[solar@routeur ~]$
```
---
🌞 Effectuer une connexion SSH sur le nouveau port
```
PS C:\Users\maxim> ssh solar@routeur -p 22000
solar@routeur's password:
Last login: Sat Dec  2 14:47:07 2023 from 10.7.1.1
[solar@routeur ~]$
```

## 3. Connexion par clé

### B. Manips
---
🌞 Générer une paire de clés
```
PS C:\Users\maxim>  ssh-keygen -t rsa -b 4096
Generating public/private rsa key pair.
Enter file in which to save the key (C:\Users\maxim/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\maxim/.ssh/id_rsa
Your public key has been saved in C:\Users\maxim/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:54J/JCnYP8n7+Quh3ERRk8m7TmvYEy1lVhRRzc/f1J0 maxim@ROGG14Max
The key's randomart image is:
+---[RSA 4096]----+
|          .ooo =B|
|           .+.  +|
|          .  . o=|
|         .  . +E*|
|     o  S.+  * .o|
|    . ooo*..= . o|
|      .++++= +   |
|       .=.oo*    |
|        o=oooo   |
+----[SHA256]-----+
```
---
🌞 Déposer la clé publique sur une VM

```
maxim@ROGG14Max MINGW64 ~
$ ssh-copy-id solar@john
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/c/Users/maxim/.ssh/id_rsa.pub"
The authenticity of host 'john (10.7.1.11)' can't be established.
ED25519 key fingerprint is SHA256:9EhlY279YtTMkCFfvAG8Xo7xKdUmdDvXO2XTyqPr2Bk.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: 10.7.1.11
    ~/.ssh/known_hosts:4: routeur
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
solar@john's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'solar@john'"
and check to make sure that only the key(s) you wanted were added.
```
---
🌞 Connectez-vous en SSH à la machine
```
PS C:\Users\maxim> ssh solar@john
Last login: Sat Dec  2 14:46:39 2023
[solar@john ~]$
```

### C. Changement de fingerprint
---
🌞 Supprimer les clés sur la machine router.tp7.b1
```
[solar@routeur ssh]$ sudo rm ssh_host_*
[sudo] password for solar:
[solar@routeur ssh]$ ls
moduli  ssh_config  ssh_config.d  sshd_config  sshd_config.d
[solar@routeur ssh]$
```
---
🌞 Regénérez les clés sur la machine router.tp7.b1
```
[solar@routeur ~]$ sudo ssh-keygen -A
ssh-keygen: generating new host keys: RSA DSA ECDSA ED25519
```
---
🌞 Tentez une nouvelle connexion au serveur
```

PS C:\Users\maxim> ssh solar@routeur -p 22000
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:I5oOZjaZkaqBu+pBhojTMJ3wJiMRCaEOxiiSOVuXfxc.
Please contact your system administrator.
Add correct host key in C:\\Users\\maxim/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in C:\\Users\\maxim/.ssh/known_hosts:8
Host key for [routeur]:22000 has changed and you have requested strict checking.
Host key verification failed.
```
> -> Add correct host key in C:\\Users\\maxim/.ssh/known_hosts to get rid of this message. On peut vider le fichier know_host pour reinitialiser.

```

PS C:\Users\maxim> ssh solar@routeur -p 22000
The authenticity of host '[routeur]:22000 ([10.7.1.254]:22000)' can't be established.
ED25519 key fingerprint is SHA256:I5oOZjaZkaqBu+pBhojTMJ3wJiMRCaEOxiiSOVuXfxc.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[routeur]:22000' (ED25519) to the list of known hosts.
solar@routeur's password:
Last login: Sat Dec  2 15:31:14 2023 from 10.7.1.1
```


## III. Web sécurisé
🌞 Montrer sur quel port est disponible le serveur web

```

[solar@web ~]$ sudo ss -alntpu
tcp   LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=1521,fd=6),("nginx",pid=1520,fd=6))
tcp   LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=678,fd=3))
tcp   LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=1521,fd=7),("nginx",pid=1520,fd=7))
tcp   LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=678,fd=4))
```

### 1. Setup HTTPS
---
🌞 Générer une clé et un certificat sur web.tp7.b1

```

[solar@web ~]$  openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt
.................+...............+....+..+....+......+...........+...+......+.+..............+....+.....+......+...+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+...........+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*...........+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*......+......+......+....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*....+..+......+.........+.+...............+......+.....+...+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:FR
State or Province Name (full name) []:Nouvelle Aquitaine
Locality Name (eg, city) [Default City]:Bordeaux
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
```

```

$ sudo mv server.key /etc/pki/tls/private/web.tp7.b1.key
$ sudo mv server.crt /etc/pki/tls/certs/web.tp7.b1.crt
$ sudo chown nginx:nginx /etc/pki/tls/private/web.tp7.b1.key
$ sudo chown nginx:nginx /etc/pki/tls/certs/web.tp7.b1.crt
$ sudo chmod 0400 /etc/pki/tls/private/web.tp7.b1.key
$ sudo chmod 0444 /etc/pki/tls/certs/web.tp7.b1.crt
```
---
🌞 Modification de la conf de NGINX

```

[solar@web ~]$  sudo cat /etc/nginx/conf.d/site_web.conf
server {
    # on change la ligne listen
    listen 10.7.1.12:443 ssl;

    # et on ajoute deux nouvelles lignes
    ssl_certificate /etc/pki/tls/certs/web.tp7.b1.crt;
    ssl_certificate_key /etc/pki/tls/private/web.tp7.b1.key;

    server_name www.site_web_nul.b1;
    root /var/www/site_web_nul;
}
```
---
🌞 Conf firewall
```

[solar@web ~]$ sudo firewall-cmd --add-port=443/tcp --permanent
success
[solar@web ~]$ sudo firewall-cmd --reload
success
```
---
🌞 Redémarrez NGINX
```

sudo systemctl restart nginx
```
---
🌞 Prouvez que NGINX écoute sur le port 443/tcp
```

[solar@web ~]$ sudo ss -alntpu
tcp   LISTEN 0      511        10.7.1.12:443       0.0.0.0:*    users:(("nginx",pid=1618,fd=6),("nginx",pid=1617,fd=6))
```
---
🌞 Visitez le site web en https
```
[solar@john ~]$ curl -k https://10.7.1.12
<h1>MEOW <3</h1>
```
	