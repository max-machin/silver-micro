# silver-micro
Projet de machine virtuelle dockerisée.   
Hyperviseur de type 2 : VMware  
Distribution : Debian 12

## Installation de base (VM + distribution)
1. Choisir un hyperviseur de type 2 (cet exemple utilise VMware)  
Installer le logiciel de virtualisation VMware workstation player : https://www.vmware.com/products/workstation-player.html

2. Télécharger l'image iso de distribution Debian basique (v12) : https://www.debian.org/download

3. À partir de l'interface VMware, créer une machine virtuelle utilisant la distribution Debian basique.

Une fois la machine virtuelle monter et démarrer, ouvrir un terminal de commande.   
Pour des questions de permissions, exécuter la commande suivante : ( cela évite le nécessité de "sudo" avant chacune des commandes à effectuer )  
```console
max123@debian:~$ sudo -i 
```

L'utilisateur devrait désormais être root : 
```console
root@debian:~$ 
```

Mise à jour des paquets :
```console
root@debian:~$ apt update
```

## Stack LAMP
#### 1. Installer Apache 
```console
root@debian:~$ apt install apache2
```
Une fois installé, activer et démarrer le service : 
```console
root@debian:~$ systemctl enable --now apache2
root@debian:~$ systemctl start apache2
```

#### 2. Installer MariaDb 
```console
root@debian:~$ apt install mariadb-server
```

Démarrer et activer le service mariaDb : 
```console
root@debian:~$ systemctl start mariadb
root@debian:~$ systemctl enable mariadb
```

Check la status du service : 
```console
root@debian:~$ systemctl status mariadb --no-pager -l
```

#### 3. Installer PHP
```console
root@debian:~$ apt install php libapache2-mod-php
```

 Créer un nouveau fichier texte :
 ```console
root@debian:~$ nano /var/www/html/info.php
```

Ajouter la ligne suivante dans le fichier : 
```console
<?php phpinfo(); ?>
```
Appuyer sur Ctrl + X, ensuite Y puis Entrée pour écrire et enregistrer le fichier.

Accèder à la page PHP depuis : 
```console
http://your_server_ip/info.php
or
http://your_domain/info.php
```

Pour récupérer l'ip serveur de votre machine, il est possible depuis le terminal de commande de la machine virtuelle d'installer le paquet : 
```console
root@debian:~$ apt install net-tools
```

Puis run :
```console
ifconfig
```

L'adresse ip cible est l'adresse inet : 
```console
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.142.128  netmask 255.255.255.0  broadcast 192.168.142.255
        inet6 fe80::20c:29ff:feaa:380a  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:aa:38:0a  txqueuelen 1000  (Ethernet)
        RX packets 30234  bytes 42499023 (40.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3842  bytes 349948 (341.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## Stack MERN 
#### 1. Installer MongoDb
Il faut installer gnupg en amont : 
```console
root@debian:~$ apt install gnupg
```

Ensuite importer les clefs publics : 
```console
wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
```

Ajouter le repo aux sources : 
```console
echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/5.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
```

Download libssl1 et l'installer : 
```console
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2_amd64.deb
sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2_amd64.deb
```

Enfin, installer mongoDb : 
```console
sudo apt-get install -y mongodb-org
```

