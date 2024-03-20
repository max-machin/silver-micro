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
max123@debian:~# sudo -i 
```

L'utilisateur devrait désormais être root : 
```console
root@debian:~# 
```

Mise à jour des paquets :
```console
root@debian:~# apt update
```

## Stack LAMP
### 1. Installer Apache 
```console
root@debian:~# apt install apache2
```
Une fois installé, activer et démarrer le service : 
```console
root@debian:~# systemctl enable --now apache2
root@debian:~# systemctl start apache2
```

### 2. Installer MariaDb 
```console
root@debian:~# apt install mariadb-server
```

Démarrer et activer le service mariaDb : 
```console
root@debian:~# systemctl start mariadb
root@debian:~# systemctl enable mariadb
```

Check la status du service : 
```console
root@debian:~# systemctl status mariadb --no-pager -l
```

### 3. Installer PHP
```console
root@debian:~# apt install php libapache2-mod-php
```

 Créer un nouveau fichier texte :
 ```console
root@debian:~# nano /var/www/html/info.php
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
root@debian:~# apt install net-tools
```

Puis run :
```console
ifconfig
```

L'adresse ip cible est l'adresse inet : 
```console
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.111.111  netmask 255.255.255.0  broadcast 192.168.142.255
        inet6 ze18::20c:32re:rftt:380a  prefixlen 64  scopeid 0x20<link>
        ether 11:0c:29:bb:23:0a  txqueuelen 1000  (Ethernet)
        RX packets 30234  bytes 42499023 (40.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3842  bytes 349948 (341.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## Stack MERN 
### 1. Installer MongoDb
Il faut installer gnupg en amont : 
```console
root@debian:~# apt install gnupg
```

Ensuite importer les clefs publics : 
```console
root@debian:~# wget -qO - https://www.mongodb.org/static/pgp/server-5.0.asc | sudo apt-key add -
```

Ajouter le repo aux sources : 
```console
root@debian:~# echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/5.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
```

Download libssl1 et l'installer : 
```console
root@debian:~# wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2_amd64.deb
root@debian:~# dpkg -i libssl1.1_1.1.1f-1ubuntu2_amd64.deb
```

Enfin, installer MongoDb : 
```console
root@debian:~# apt-get install -y mongodb-org
```

Activer MongoDb : 
```console
root@debian:~# systemctl enable mongod
```

Start le server : 
```console
root@debian:~# service mongod start
```

Check le status pour être sur que le serveur run :
```console
root@debian:~# service mongod status
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; preset: enabl>
     Active: active (running) since Tue 2024-03-19 04:28:06 CET; 24s ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 5453 (mongod)
     Memory: 76.3M
        CPU: 956ms
     CGroup: /system.slice/mongod.service
             └─5453 /usr/bin/mongod --config /etc/mongod.conf

```

### 2. Configurer MongoDb
Editer le fichier de config : 
```console
root@debian:~# nano /etc/mongod.conf
```
Décommenter la partie #security, au final il doit ressemble à ça : 
```console
security:
  authorization: enabled
```
Puis autoriser les connexions externes(Pour récupérer l'ip, se référer : ###3. Installer PHP) : 
```console
net:
  port: 27017
  bindIp: 127.0.0.1,[your_ip]
```

Restart le server : 
```console
root@debian:~# systemctl restart mongod
```

Confirmer que MongoDb autorise les connexions externes : 
```console
root@debian:~# lsof -i | grep mongo
mongod    5605  mongodb   12u  IPv4  41849      0t0  TCP localhost:27017 (LISTEN)
mongod    5605  mongodb   13u  IPv4  41850      0t0  TCP debian-MERN:27017 (LISTEN)
```

Créer un admin MongoDb : 
```console
root@debian:~# mongosh
Current Mongosh Log ID:	65f90a4c24f757d7fedb83af
Connecting to:		mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.2.0
Using MongoDB:		5.0.25
Using Mongosh:		2.2.0

root@debian:~# use admin
switched to db admin
```

Créer un admin avec tous les rôles et privilèges et créer un password : 
```console
root@debian:~# db.createUser({user: "admin" , pwd: passwordPrompt() , roles: [{ role: "userAdminAnyDatabase" , db: "admin"}]})
Enter password
******{ ok: 1 }
```

Maintenant, il est possible d'utiliser la string suivante pour se connecter à MongoDb : 
```console
mongodb://admin:password@External-IP:27017/database
```

### 3. Installer Node avec NVM
L'installation de NVM nécessite les packages suivants, vérifier qu'ils soient bien présent sur la machine virtuelle : 
```console
root@debian:~# apt install build-essential libssl-dev
root@debian:~# apt-get install curl 
```

Installer NVM : 
```console
root@debian:~# curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash 
```

Activer l'environnement pour la première fois 
```console
root@debian:~# source ~/.bashrc
```

Vérifier que l'installation est réussie, fermer le terminal ouvrer un nouveau et essayer cette commande : 
```console
root@debian:~# command -v nvm 
```

Il est possible de changer la version utiliser à l'aide de cette commande : 
```console
root@debian:~# nvm use 18 
```

Pour vérifier la version installer : 
```console
root@debian:~# node -v
v18.19.1
root@debian:~# npm -v
10.2.4
```

### 3. Installer React.js
Placez-vous dans le dossier ou vous souhaiter installer l'app puis à l'aide de la commande ci-dessous, l'installer : 
```console
root@debian:~# npx create-react-app frontend
```

Se rendre dans le dossier de l'application et run un build : 
```console
root@debian:~# cd [your_folder]/frontend
root@debian:~# npm run build
```

### 4. Installer Express.js

installer express-generator :
```console
root@debian:~# npm install -g express-generator
```

Créer une app backend : 
```console
root@debian:~# express --view=pug backend
```

Installer les dépendances : 
```console
root@debian:~# cd [your_folder]/backend
root@debian:~# npm install
```

Pour run l'app : 
MacOS ou Linux 
```console
$ DEBUG=myapp:* npm start
```
Windows cmd prompt : 
```console
> set DEBUG=myapp:* & npm start
```
Windows Powershell
```console
PS> $env:DEBUG='myapp:*'; npm star
```

Accèder à l'url suivante : 
http://localhost:3000/

## Docker 
### 1. Container Php & Apache
#### 1.1 Création du container 

Créer un dockerfile dans le dossier de votre projet.  
Il faut noter que le dossier src devra être le dossier racine de l'application contenant tout le code PHP
Image php:7.2-apache : 
```dockerfile
# Dockerfile
FROM php:7.2-apache
COPY src/ /var/www/html/
```
Build l'image : 
```dockerfile
docker build -t my-php-app .
```
Run le container sur le port 80 : 
```dockerfile
docker run -d -p 80:80 --name my-running-app my-php-app
```

L'app devrait alors être disponible depuis http://localhost:80
