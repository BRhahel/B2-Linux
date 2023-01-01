# TP4 : Conteneurs

Dans ce TP on va aborder plusieurs points autour de la conteneurisation : 

- Docker et son empreinte sur le système
- Manipulation d'images
- `docker-compose`

# Sommaire

- [TP4 : Conteneurs](#tp4--conteneurs)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
  - [Checklist](#checklist)
- [I. Docker](#i-docker)
  - [1. Install](#1-install)
  - [2. Vérifier l'install](#2-vérifier-linstall)
  - [3. Lancement de conteneurs](#3-lancement-de-conteneurs)
- [II. Images](#ii-images)
- [III. `docker-compose`](#iii-docker-compose)
  - [1. Intro](#1-intro)
  - [2. Make your own meow](#2-make-your-own-meow)

# I. Docker

🖥️ Machine **docker1.tp4.linux**

## 1. Install

🌞 **Installer Docker sur la machine**

```
[rhahel@localhost ~]$ sudo dnf check-update
[rhahel@localhost ~]$ sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
[rhahel@localhost ~]$ sudo dnf install docker-ce docker-ce-cli containerd.io
```
- Démarrer le service

```
[rhahel@localhost ~]$ sudo systemctl start docker
[root@localhost rhahel]# sudo usermod -aG docker $(whoami)
```

## 2. Vérifier l'install

➜ **Vérifiez que Docker est actif est disponible en essayant quelques commandes usuelles :**

```
[root@localhost rhahel]# sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
     Active: active (running) since Thu 2022-11-24 11:36:24 CET; 10min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 816 (dockerd)
      Tasks: 7
     Memory: 97.2M
        CPU: 184ms
     CGroup: /system.slice/docker.service
             └─816 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

➜ **Explorer un peu le help**, si c'est pas le man :

```
[root@localhost rhahel]# docker run --help
```

## 3. Lancement de conteneurs

🌞 **Utiliser la commande `docker run`**

- lancer un conteneur `nginx`

  - l'app NGINX doit avoir un fichier de conf personnalisé
  - l'app NGINX doit servir un fichier `index.html` personnalisé
  - l'application doit être joignable grâce à un partage de ports
  - vous limiterez l'utilisation de la RAM et du CPU de ce conteneur
  - le conteneur devra avoir un nom
  - le processus exécuté par le conteneur doit être un utilisateur de votre choix (pas `root`)

```
[root@localhost toto]# docker run -p 17777:9999 -v /home/rhahel/toto/index.html:/var/www/tp4/index.html  -v /home/rhahel/toto/toto.conf:/etc/nginx/conf.d/to
to.conf -m 512M nginx
/docker-entrypoint.sh: Configuration complete; ready for start up
```

# II. Images

La construction d'image avec Docker est basée sur l'utilisation de fichiers `Dockerfile`.

L'idée est la suivante :

- vous créez un dossier de travail
```
[rhahel@localhost ~]$ mkdir doc
```
- vous vous déplacez dans ce dossier de travail
- vous créez un fichier `Dockerfile`
  - il contient les instructions pour construire une image
  - `FROM` : indique l'image de base
  - `RUN` : indique des opérations à effectuer dans l'image de base
```
[rhahel@localhost doc]$ cat Dockerfile
FROM Rocky Network

RUN apt update -y

RUN apt install -y nginx
```
- vous exécutez une commande `docker build . -t <IMAGE_NAME>`
```
[rhahel@localhost doc]$ docker build . -t test_zz
Successfully built 3dcaae517ae4
Successfully tagged test_zz:latest
```
- une image est produite, visible avec la commande `docker images`
```
[rhahel@localhost doc]$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
test_zz      latest    3dcaae517ae4   8 minutes ago   211MB
nginx        latest    88736fe82739   9 days ago      142MB
debian       latest    c31f65dd4cc9   10 days ago     124MB
```

## 2. Construisez votre propre Dockerfile

🌞 **Construire votre propre image**

- image de base (celle que vous voulez : debian, alpine, ubuntu, etc.)
  - une image du Docker Hub
  - qui ne porte aucune application par défaut
- vous ajouterez
  - mise à jour du système
  - installation de Apache
  - page d'accueil Apache HTML personnalisée
