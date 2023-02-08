# Sujet Rattrapage B2

## Sommaire

- [Sujet Rattrapage B2](#sujet-rattrapage-b2)
  - [Sommaire](#sommaire)
  - [Checklist](#checklist)
- [0. Le lab](#0-le-lab)
- [I. Setup](#i-setup)
  - [1. Emby server](#1-emby-server)
  - [2. Reverse proxy](#2-reverse-proxy)
    - [A. Setup](#a-setup)
    - [B. Secu](#b-secu)
  - [3. Backup](#3-backup)
    - [A. Serveur NFS](#a-serveur-nfs)
    - [B. Client NFS](#b-client-nfs)

 I. Setup

## 1. Emby server

> A faire sur 🖥️ `web.peche.linux`.

🌞 **Mettez en place le serveur Emby en suivant la doc officielle**

```
[rhahel@localhost ~]$ sudo yum install https://github.com/MediaBrowser/Emby.Releases/releases/download/4.7.11.0/emby-server-rpm_4.7.11.0_x86_64.rpm
[...]
Installed:
  emby-server-4.7.11.0-1.x86_64

Complete!
```

🌞 **Une fois en place, mettez en évidence :**
- quel service est démarré :
```
[rhahel@localhost ~]$ systemctl status emby-server
```
- comment consulter les logs de Emby :
```
[rhahel@localhost ~]$ journalctl -u emby-server
```
- quel processus sont lancés par ce service : 
```
[rhahel@localhost ~]$ ps -ef | grep emby
emby         689       1  0 21:12 ?        00:00:04 /opt/emby-server/system/EmbyServer -programdata /var/lib/emby -ffdetect /opt/emby-server/bin/ffdetect -ffmpeg /opt/emby-server/bin/ffmpeg -ffprobe /opt/emby-server/bin/ffprobe -restartexitcode 3 -updatepackage emby-server-rpm_{version}_x86_64.rpm
root         964     890  0 21:19 pts/0    00:00:00 sudo journalctl -u emby-server
root         966     964  0 21:19 pts/0    00:00:00 journalctl -u emby-server
rhahel       996     890  0 21:23 pts/0    00:00:00 grep --color=auto emby
```
- sur quel(s) port(s) ce(s) processus écoute(nt) : 
```
[rhahel@localhost ~]$ ss -plunt | grep Emby
```
- avec quel utilisateur est lancé le(s) processus :
```
[rhahel@localhost ~]$ ps -ef | grep emby | awk '{print $1}'
emby
root
root
rhahel
```

🌞 **Accédez à l'interface Web**

- il sera nécessaire d'ouvrir un port firewall
```
[root@localhost rhahel]# firewall-cmd --add-port=8096/tcp --permanent
success
[root@localhost rhahel]# firewall-cmd --reload
success
```
- vous y accéder depuis le navigateur de votre PC avec : `http://10.102.1.11:8096/`
- vous pourrez y finaliser l'install

🌞 **Configurer un répertoire pour accueillir vos médias**

- créer un répertoire `/srv/media/`
```
[root@localhost rhahel]# mkdir /srv/media/
```
- faites le appartenir au bon user (celui qui lance le serveur Emby : c'est lui qui aura besoin d'accéder aux données)
```
[root@localhost rhahel]# chown emby:emby /srv/media/
```
- donnez lui les permissions les plus restrictives possibles, le minimumm pour que tout fonctionne correctement
```
chmod 700 /srv/media
```
- uploadez un morceau de musique sur la VM (depuis votre PC) avec la commande `scp`
```
[root@localhost rhahel]# scp C:\\Users\\rhahe\\Music\\emby root@10.102.1.11:/srv/media/
```
- sur l'interface Web de Emby, indiquez que vous avez une nouvelle bibliothèque musicale dans `/srv/media/`
- vous devriez pouvoir écouter votre morceau depuis la WebUI

## 2. Reverse proxy

### A. Setup

> A faire sur 🖥️ `proxy.peche.linux`.

🌞 **Mettez en place NGINX**

- On commence par installer NGINIX :
```
[root@localhost rhahel]# dnf install nginx
```
```
[root@localhost rhahel]# nano /etc/nginx/conf.d/emby.conf
[root@localhost rhahel]# cat /etc/nginx/conf.d/emby.conf
server {

    server_name proxy.peche.linux;
    listen 80;
    listen 443 ssl;
    ssl_certificate /etc/nginx/certificate/nginx-certificate.crt;
    ssl_certificate_key /etc/nginx/certificate/nginx.key;


    location / {
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-Host $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

        # On définit la cible du proxying
        proxy_pass http://10.102.1.10:8096;
    }

    # Deux sections location recommandés par la doc NextCloud
    location /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
}

```

### B. Secu

> A faire sur 🖥️ `web.peche.linux`.

🌞 **Ajoutez une règle firewall**


## 3. Backup

### A. Serveur NFS

> A faire sur 🖥️ `backup.peche.linux`.

🌞 **Mettez en place un serveur NFS**

- création d'un répertoire `/backups`
```
[root@localhost rhahel]# mkdir /backups
```
- création d'un répertoire `/backups/music/`
```
[root@localhost rhahel]# mkdir /backups/music/
```
- référez-vous au [à la partie II du module 4 du TP3](../3/4-backup/README.md#ii-nfs) pour + de détails
  - vous devez partager à la machine `web.peche.linux` le dossier `/backup/music/`

```
[root@localhost music]# useradd backup
[root@localhost /]# nano /etc/exports
[root@localhost /]# cat /etc/exports
/backups/music proxy.peche.linux(rw,sync,no_root_squash,no_all_squash)
```
### B. Client NFS

> A faire sur 🖥️ `web.peche.linux`.

🌞 **Mettez en place un client NFS**

- création d'un répertoire `/backup`
```
[root@localhost rhahel]# mkdir /backups
```
- référez-vous au [à la partie II du module 4 du TP3](../3/4-backup/README.md#ii-nfs) pour + de détails
  - vous devez monter le dossier `/backups/music/` qui se trouve sur `backup.peche.linux`
  - et le rendre accessible sur le dossier `/backup`
