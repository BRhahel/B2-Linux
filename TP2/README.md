# TP2 : Gestion de service

# Sommaire

- [TP2 : Gestion de service](#tp2--gestion-de-service)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
  - [Checklist](#checklist)
- [I. Un premier serveur web](#i-un-premier-serveur-web)
  - [1. Installation](#1-installation)
  - [2. Avancer vers la maîtrise du service](#2-avancer-vers-la-maîtrise-du-service)
- [II. Une stack web plus avancée](#ii-une-stack-web-plus-avancée)
  - [1. Intro blabla](#1-intro-blabla)
  - [2. Setup](#2-setup)
    - [A. Base de données](#a-base-de-données)
    - [B. Serveur Web et NextCloud](#b-serveur-web-et-nextcloud)
    - [C. Finaliser l'installation de NextCloud](#c-finaliser-linstallation-de-nextcloud)

# I. Un premier serveur web

## 1. Installation

🖥️ **VM web.tp2.linux**

| Machine         | IP            | Service     |
|-----------------|---------------|-------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web |

🌞 **Installer le serveur Apache**

```
[root@localhost /]# dnf install httpd
[root@localhost /]# sudo vim :g/^ *#.*/d /etc/httpd/conf/httpd.conf
```

🌞 **Démarrer le service Apache**

```
[root@localhost ~]# sudo systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
[root@localhost ~]# sudo systemctl start httpd

[root@localhost ~]# sudo ss -alnpt
State           Recv-Q          Send-Q                   Local Address:Port                   Peer Address:Port         Process
LISTEN          0               128                            0.0.0.0:22                          0.0.0.0:*
 users:(("sshd",pid=685,fd=3))
LISTEN          0               511                                  *:80                                *:*
 users:(("httpd",pid=1355,fd=4),("httpd",pid=1354,fd=4),("httpd",pid=1353,fd=4),("httpd",pid=1351,fd=4))
LISTEN          0               128                               [::]:22                             [::]:*
 users:(("sshd",pid=685,fd=4))
[root@localhost ~]# sudo firewall-cmd --add-port=80/tcp --permanent
success
```

🌞 **TEST**

- vérifier que le service est démarré

```

```
- vérifier qu'il est configuré pour démarrer automatiquement

```

```
- vérifier avec une commande `curl localhost` que vous joignez votre serveur web localement

```
[rhahel@localhost ~]$ curl 10.102.1.11
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
      }
```
- vérifier avec votre navigateur (sur votre PC) que vous accéder à votre serveur web

```
PS C:\Users\rhahe> curl 10.102.1.11
curl : HTTP Server Test Page
This page is used to test the proper operation of an HTTP server after it has been installed on a Rocky Linux system.
If you can read this page, it means that the software it working correctly.
```

## 2. Avancer vers la maîtrise du service

🌞 **Le service Apache...**

- Contenu du fichier `httpd.service`

```
[rhahel@localhost system]$ cat sshd.service
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.target
Wants=sshd-keygen.target

[Service]
Type=notify
EnvironmentFile=-/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```
🌞 **Déterminer sous quel utilisateur tourne le processus Apache**

```
[rhahel@localhost ~]$ cd /etc/httpd/conf/
[rhahel@localhost conf]$ cat httpd.conf
User apache
Group apache
```
```
[rhahel@localhost conf]$ ps -ef
apache       707     684  0 11:33 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       710     684  0 11:33 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       711     684  0 11:33 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       712     684  0 11:33 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
```
- Verifier que le contenu est accessible en lecture
```
[rhahel@localhost ~]$ cd /usr/share/testpage/
[rhahel@localhost testpage]$ ls -al
total 12
drwxr-xr-x.  2 root root   24 Nov 17 12:01 .
drwxr-xr-x. 82 root root 4096 Nov 17 12:01 ..
-rw-r--r--.  1 root root 7620 Jul  6 04:37 index.html
```

🌞 **Changer l'utilisateur utilisé par Apache**

- Création d'un nouvel utilisateur

```
[rhahel@localhost ~]$ cat /etc/passwd | grep "apache"
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
papache:x:1001:1001::/home/papache:/bin/bash
[rhahel@localhost ~]$ sudo useradd -d /usr/share/httpd -s /sbin/nologin -U pache
[sudo] password for rhahel:
useradd: warning: the home directory /usr/share/httpd already exists.
useradd: Not copying any file from skel directory into it.
```
- Modifiez la configuration d'Apache pour qu'il utilise ce nouvel utilisateur

```
[rhahel@localhost ~]$ sudo vi /etc/httpd/conf/httpd.conf
[...]
User pache
Group pache
[...]
```

- redémarrez Apache

```
[rhahel@localhost ~]$ sudo systemctl restart httpd
```

- utilisez une commande `ps` pour vérifier que le changement a pris effet

```
[rhahel@localhost ~]$ ps -ef
pache       1793    1792  0 04:12 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
pache       1794    1792  0 04:12 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
pache       1795    1792  0 04:12 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
pache       1796    1792  0 04:12 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
```

🌞 **Faites en sorte que Apache tourne sur un autre port**

- Modifiez la configuration d'Apache pour lui demander d'écouter sur un autre port de votre choix

```
[rhahel@localhost ~]$ sudo firewall-cmd --remove-port=80/tcp --permanent
[sudo] password for rhahel:
success
```
- Ouvrez ce nouveau port dans le firewall, et fermez l'ancien

```
[rhahel@localhost ~]$ sudo firewall-cmd --add-port=8080/tcp --permanent
success
```

- Redémarrer Apache

```
[rhahel@localhost ~]$ sudo systemctl restart httpd
```

- prouvez avec une commande `ss` que Apache tourne bien sur le nouveau port choisi

```
[rhahel@localhost ~]$ ss -alnpt
State         Recv-Q        Send-Q               Local Address:Port               Peer Address:Port       Process
LISTEN        0             128                        0.0.0.0:22                      0.0.0.0:*
LISTEN        0             511                              *:80                            *:*
LISTEN        0             128                           [::]:22                         [::]:*
```

- `curl`

```
PS C:\Users\rhahe> curl 10.102.1.11:80
curl : HTTP Server Test Page
This page is used to test the proper operation of an HTTP server after it has been installed on a Rocky Linux system. If you can read this page, it means
that the software it working correctly.
```

📁 **Fichier `/etc/httpd/conf/httpd.conf`**

[Le fichier est accesible à ce lien](httpd.conf)

# II. Une stack web plus avancée

## 1. Intro blabla

