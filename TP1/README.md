# TP1 : (re)Familiaration avec un système GNU/Linux

## Sommaire

- [TP1 : (re)Familiaration avec un système GNU/Linux](#tp1--refamiliaration-avec-un-système-gnulinux)
  - [Sommaire](#sommaire)
  - [0. Préparation de la machine](#0-préparation-de-la-machine)
  - [I. Utilisateurs](#i-utilisateurs)
    - [1. Création et configuration](#1-création-et-configuration)
    - [2. SSH](#2-ssh)
  - [II. Partitionnement](#ii-partitionnement)
    - [1. Préparation de la VM](#1-préparation-de-la-vm)
    - [2. Partitionnement](#2-partitionnement)
  - [III. Gestion de services](#iii-gestion-de-services)
  - [1. Interaction avec un service existant](#1-interaction-avec-un-service-existant)
  - [2. Création de service](#2-création-de-service)
    - [A. Unité simpliste](#a-unité-simpliste)
    - [B. Modification de l'unité](#b-modification-de-lunité)

## 0. Préparation de la machine

🌞 **Ajouter un utilisateur** à la machine, qui sera dédié à son administration
```
[root@localhost /]# useradd toto -m -s /bin/bash
```
```
[root@localhost /]# useradd toto -m -s /bin/bash
```

🌞 **Créer un nouveau groupe `admins`** qui contiendra les utilisateurs de la machine ayant accès aux droits de `root` *via* la commande `sudo`.

```
[root@localhost /]# groupadd admins
[root@localhost /]# cd /etc/
[root@localhost etc]# visudo sudoers
```
🌞 **Ajouter votre utilisateur à ce groupe `admins`**

```
[root@localhost /]# usermod -aG admins toto
```

### 2. SSH

Afin de se connecter à la machine de façon plus sécurisée, on va configurer un échange de clés SSH lorsque l'on se connecte à la machine.

🌞 **Pour cela...**

- il faut générer une clé sur le poste client de l'administrateur qui se connectera à distance (vous :) )
  - génération de clé depuis VOTRE poste donc
  - avec la commande `ssh-keygen` (avec des options)
- déposer la clé dans le fichier `/home/<USER>/.ssh/authorized_keys` de la machine que l'on souhaite administrer
  - vous utiliserez l'utilisateur que vous avez créé dans la partie précédente du TP
  - on peut le faire à la main (voir le cours SSH)
  - ou avec la commande `ssh-copy-id` (dispo uniquement dans `git bash` si vous êtes sous Windows)

```
[root@localhost /]# ssh-keygen
```

🌞 **Assurez vous que la connexion SSH est fonctionnelle**, sans avoir besoin de mot de passe.

🌞 **Assurez vous que la connexion SSH est fonctionnelle**, sans avoir besoin de mot de passe.

## II. Partitionnement

### 1. Préparation de la VM

Ajout de deux disques durs à la machine virtuelle, de 3Go chacun.

🌞 **Utilisez LVM** pour...

- agréger les deux disques en un seul *volume group*
- créer 3 *logical volumes* de 1 Go chacun
- formater ces partitions en `ext4`
- monter ces partitions pour qu'elles soient accessibles aux points de montage `/mnt/part1`, `/mnt/part2` et `/mnt/part3`.

🌞 **Grâce au fichier `/etc/fstab`**, faites en sorte que cette partition soit montée automatiquement au démarrage du système.

### 2. Partitionnement

- Les disques à ajouter :
```
[toto@node1 ~]$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
[...]
sdb           8:16   0    3G  0 disk
sdc           8:32   0    3G  0 disk
[...]

```

- On ajoute les disques PV (Physical Volume)
```
[toto@node1 ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[toto@node1 ~]$ sudo pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.

[toto@node1 ~]$ sudo pvs
  PV         VG Fmt  Attr PSize PFree
  /dev/sdb      lvm2 ---  3.00g 3.00g
  /dev/sdc      lvm2 ---  3.00g 3.00g

[toto@node1 ~]$ sudo pvdisplay
  "/dev/sdb" is a new physical volume of "3.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb
  PV Size               3.00 GiB
  [...]

  "/dev/sdc" is a new physical volume of "3.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc
  PV Size               3.00 GiB
  [...]

```

- Aggréer les diques en un seul volume

```
[toto@node1 ~]$ sudo vgcreate data /dev/sdb
  Volume group "data" successfully created

[toto@node1 ~]$ sudo vgextend data /dev/sdc
  Volume group "data" successfully extended

[toto@node1 ~]$ sudo vgs
  VG   #PV #LV #SN Attr   VSize VFree
  data   2   0   0 wz--n- 5.99g 5.99g

[toto@node1 ~]$ sudo vgdisplay
  --- Volume group ---
  VG Name               data
  [...]
  VG Size               5.99 GiB
  PE Size               4.00 MiB
  [...]

```
- créer 3 *logical volumes* de 1 Go chacun

```
[toto@node1 ~]$ sudo lvcreate -L 1G data -n part1
  Logical volume "part1" created.
[toto@node1 ~]$ sudo lvcreate -L 1G data -n part2
  Logical volume "part2" created.
[toto@node1 ~]$ sudo lvcreate -L 1G data -n part3
  Logical volume "part3" created.

[toto@node1 ~]$ sudo lvs
  LV    VG   Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  part1 data -wi-a----- 1.00g
  part2 data -wi-a----- 1.00g
  part3 data -wi-a----- 1.00g

[toto@node1 ~]$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/data/part1
  LV Name                part1
  VG Name                data
  [...]
  LV Size                1.00 GiB
  [...]

  --- Logical volume ---
  LV Path                /dev/data/part2
  LV Name                part2
  VG Name                data
  [...]
  LV Size                1.00 GiB
  [...]

  --- Logical volume ---
  LV Path                /dev/data/part3
  LV Name                part3
  VG Name                data
  [...]
  LV Size                1.00 GiB
  [...]

```
- formater ces partitions en `ext4`

```
  [toto@localhost ~]$ sudo mkfs -t ext4 /dev/data/part1
    mke2fs 1.46.5 (30-Dec-2021)
    Creating filesystem with 262144 4k blocks and 65536 inodes
    Filesystem UUID: d64ddc9a-34d2-4386-9b7c-4fece3c3fdd6
    Superblock backups stored on blocks:
            32768, 98304, 163840, 229376

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (8192 blocks): done
    Writing superblocks and filesystem accounting information: done

  [toto@localhost ~]$ sudo mkfs -t ext4 /dev/data/part2
    mke2fs 1.46.5 (30-Dec-2021)
    Creating filesystem with 262144 4k blocks and 65536 inodes
    Filesystem UUID: 0ff8d684-eedf-47a2-897c-7dfa0d804273
    Superblock backups stored on blocks:
            32768, 98304, 163840, 229376

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (8192 blocks): done
    Writing superblocks and filesystem accounting information: done

  [toto@localhost ~]$ sudo mkfs -t ext4 /dev/data/part3
    mke2fs 1.46.5 (30-Dec-2021)
    Creating filesystem with 262144 4k blocks and 65536 inodes
    Filesystem UUID: 3cba9538-6f50-44a4-bc66-3132094453bc
    Superblock backups stored on blocks:
            32768, 98304, 163840, 229376

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (8192 blocks): done
    Writing superblocks and filesystem accounting information: done

```

- monter ces partitions pour qu'elles soient accessibles aux points de montage `/mnt/part1`, `/mnt/part2` et `/mnt/part3`.

```
[toto@localhost ~]$ sudo mkdir /mnt/part1 && sudo mkdir /mnt/part2 && sudo mkdir /mnt/part3
[toto@localhost ~]$ ls /mnt
part1  part2  part3

[toto@localhost ~]$ sudo mount /dev/data/part1 /mnt/part1
[toto@localhost ~]$ sudo mount /dev/data/part2 /mnt/part2
[toto@localhost ~]$ sudo mount /dev/data/part3 /mnt/part3

[toto@localhost ~]$ df -h
  Filesystem              Size  Used Avail Use% Mounted on
  [...]
  /dev/mapper/data-part1  974M   24K  907M   1% /mnt/part1
  /dev/mapper/data-part2  974M   24K  907M   1% /mnt/part2
  /dev/mapper/data-part3  974M   24K  907M   1% /mnt/part3

```

🌞 **Grâce au fichier `/etc/fstab`**, faites en sorte que cette partition soit montée automatiquement au démarrage du système.

```
  [toto@localhost ~]$ sudo nano /etc/fstab
  [toto@localhost ~]$ cat /etc/fstab | grep "part"
  /dev/data/part1 /mnt/part1      ext4    defaults        0 0
  /dev/data/part2 /mnt/part2      ext4    defaults        0 0
  /dev/data/part3 /mnt/part3      ext4    defaults        0 0

  [toto@localhost ~]$ sudo umount /mnt/part1
  [toto@localhost ~]$ sudo umount /mnt/part2
  [toto@localhost ~]$ sudo umount /mnt/part3
  [toto@localhost ~]$ sudo mount -av
    [...]
    /mnt/part1               : successfully mounted
    /mnt/part2               : successfully mounted
    /mnt/part3               : successfully mounted

```

## III. Gestion de services


## 1. Interaction avec un service existant

🌞 **Assurez-vous que...**

- l'unité est démarrée

```
[toto@node1 ~]$ systemctl is-active firewalld
active
```
- l'unitée est activée (elle se lance automatiquement au démarrage)

```
[toto@node1 ~]$ systemctl is-enabled firewalld
enabled
```

## 2. Création de service

### A. Unité simpliste

🌞 **Créer un fichier qui définit une unité de service** 

```
  [toto@node1 ~]$ sudo nano /etc/systemd/system/web.service
  [toto@node1 ~]$ cat /etc/systemd/system/web.service
  [Unit]
  Description=Very simple web service

  [Service]
  ExecStart=/usr/bin/python3 -m http.server 8888

  [Install]
  WantedBy=multi-user.target

  [toto@node1 ~]$ sudo systemctl daemon-reload

  [toto@node1 ~]$ sudo firewall-cmd --add-port=8888/tcp --permanent
  success
  [ ~]$ sudo firewall-cmd --reload
  success

  [toto@node1 ~ ~]$ sudo systemctl status web
  ○ web.service - Very simple web service
      Loaded: loaded (/etc/systemd/system/web.service; disabled; vendor preset: disabled)
      Active: inactive (dead)
  [toto@node1 ~ ~]$ sudo systemctl start web
  [toto@node1 ~ ~]$ sudo systemctl enable web
  Created symlink /etc/systemd/system/multi-user.target.wants/web.service → /etc/systemd/system/web.service.

```

🌞 **Une fois le service démarré, assurez-vous que pouvez accéder au serveur web**

```
#Sur node2

  [toto@node1 ~]$ curl 10.101.1.11:8888
  <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
  <html>
    [...]
  </html>
```

### B. Modification de l'unité

🌞 **Préparez l'environnement pour exécuter le mini serveur web Python**

- Création d'un utilisateur *web*

```
# Création de l'utilisateur
[toto@node1 ~]$ sudo useradd web

# Vérification
[toto@node1 ~]$ sudo cat /etc/passwd | grep "web"
web:x:1002:1003::/home/web:/bin/bash
```

- Création d'un dossier */var/www/meow*

```
# Création du dossier
[toto@node1 ~]$ sudo mkdir /var/www
[toto@node1 ~]$ sudo mkdir /var/www/meow

# Don de la propriété du dossier à l'utilisateur web
[toto@node1 ~]$ sudo chown web:web /var/www/meow

# Verification
[toto@node1 ~]$ ls -al /var/www | grep "meow"
drwxr-xr-x.  2 web  web     6 Nov 14 19:56 meow

```

- Création d'un fichier dans */var/www/meow*

```
# Passage en utilisateur web
[toto@node1 ~]$ sudo su - web

# Création du fichier
[web@node1 ~]$ echo "MEOW" | touch /var/www/meow/meow.html

```

- Affichage des permissions du dossier */var/www/meow* et son contenu

```
[web@node1 ~]$ ls -al /var/www/meow
total 0
drwxr-xr-x. 2 web  web  23 Nov 14 20:00 .           # Permissions du dossier
drwxr-xr-x. 3 root root 18 Nov 14 19:56 ..
-rw-r--r--. 1 web  web   0 Nov 14 20:00 meow.html   # Permissions du fichier
```

🌞 **Modifiez l'unité de service web.service créée précédemment en ajoutant les clauses**

```
  # Écriture des changements
  [toto@node1 ~]$ sudo nano /etc/systemd/system/web.service

  # Vérification
  [toto@node1 ~]$ cat /etc/systemd/system/web.service
  [Unit]
  Description=Very simple web service

  [Service]
  ExecStart=/usr/bin/python3 -m http.server 8888
  User=web
  WorkingDirectory=/var/www/meow/

  [Install]
  WantedBy=multi-user.target

  # Rechargement de la conf des services
  [toto@node1 ~]$ sudo systemctl daemon-reload
  [toto@node1 ~]$ sudo systemctl restart web
```

🌞 **Vérifiez le bon fonctionnement avec une commande `curl`**

```
  [toto@node2 ~]$ curl 10.101.1.11:8888
  [...]
  <li><a href="meow.html">meow.html</a></li>
  [...]

  # On retrouve bien notre fichier meow.html, donc on est dans le bon directory !
```