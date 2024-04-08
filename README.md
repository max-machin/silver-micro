# silver-micro

Projet de machine virtuelle LAMP & MERN / Container LAMP & MERN / Kubernetes   
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

## Docker ( LAMP & MERN ) 

## LAMP
### 1. Container Php & Apache

#### 1.1 Création du container 

( Possibilité de se référer sur le hub docker : https://hub.docker.com/_/php )

Créer un dockerfile dans le dossier de votre projet.  
Il faut noter que le dossier src devra être le dossier racine de l'application contenant tout le code PHP  
Image php:7.2-apache : 
```dockerfile
# Dockerfile
# Layer 1: Utilisation d'une image php:7.2-apache pour la base du container.
FROM php:7.2-apache
# Layer 2: Copier le contenu de l'app `src` dans le dossier /var/www/html du container
COPY src/ /var/www/html/
```
Exemple de dossier : 
```bash 
├── src
│   ├── index.php
│   └── home.php
├── Dockerfile
└── README.md
```

Créez un dossier src avec un fichier index.php contenant : 
```bash 
<?php
for($i=1;$i<=100;$i++) {
    $out = ($i%3?'':'Fizz').($i%5?'':'Buzz');
    echo ($out?$out:$i)."\n";
}
```

Build l'image : 
```dockerfile
docker build -t my-php-app .
```
Run le container sur le port 80 : 
```dockerfile
docker run --name my-running-app -d -p 80:80 -v "/$(pwd)/src/index.php:/var/www/html/index.php/" my-php-app
```

L'app devrait alors être disponible depuis : http://localhost:80

### 2. Container MySQL

#### 2.1 Création d'une instance SQL depuis l'image 

( Possibilité de se référer sur le hub docker : https://hub.docker.com/_/mysql )

Démarrer une instance MySQL (code approprié au cas) : 
```bash
docker run --name mysql -d \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=change-me \
    --restart unless-stopped \
    mysql:8
```
La commande démarre un container avec mySQL8. Le mot de passe pour l'utilisateur root devra être renseigné manuellement.  
Le flag -d indique que le container run en brackground jusqu'à être arrêter. Le restart indique à docker de restart sans arrêt le container.  
Le flag -p autorise la redirection de port dans le container afin de pouvoir accéder à la base de données depuis le port 3306.

#### 2.2 Persister la data avec un volume

L'image Docker MySQL est configuré de base pour stocker les données dans le dossier : /var/lib/mysql
Monter un volume dans ce dossier permet de persister le stockage de données.

Stop et supprimer l'instance SQL en cours : 
```bash
docker stop mysql
docker rm mysql
```

Puis démarrer un nouveau avec les nouvelles configurations : 
```bash
docker run --name mysql -d \
    -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=change-me \
    -v mysql:/var/lib/mysql \
    mysql:8
```
Cette commande va créer un volume Docker nommé : mysql.

Pour détruire le volume : 
```bash
docker volume rm mysql
```

### 3. Container PhpMyAdmin

#### 3.1 Création d'une instance PhpMyAdmin depuis l'image

( Possibilité de se référer sur le hub docker : https://hub.docker.com/_/phpmyadmin )

Lancer un container depuis l'image phpmyadmin liée au container MySQL précedemment lancé : 
```bash
docker run --name phpmyadmin -d --link mysql:db -p 8080:80 phpmyadmin
```

A ce moment, l'interface PhpMyAdmin est accessible depuis : http://localhost:8080/
L'utilisateur de base est root et le mot de passe est celui renseigné plus haut au démarrage du container MySQL.

## MERN

Pour pourvoir connecter les différents container, il faut mettre en place un network : 
```bash 
docker network create my-network 
```

### 1. Backend (Node Express)

Créer un dossier nommé : `backend`.
Dans ce dossier créer un fichier nommé : server.js (ce fichier contiendra le code node)  

Ouvrir un terminal, se rendre dans le repertoire `backend` et run : 
```bash 
npm init --y
```
Cette commande va permettre d'initialiser l'app et créer un package.json

Installer express dans `backend`: 
```bash 
npm install express --save
```

Dans le fichier server.js ajouter ce code : 
```js 
const express = require('express');

//Create an app
const app = express();
app.get('/', (req, res) => {
    res.send('Hello world\n');
});

//Listen port
const PORT = 8080;
app.listen(PORT);
console.log(`Running on port ${PORT}`);
```

Run l'app pour s'assurer qu'elle est fonctionnelle : 
```bash
node server.js
```

L'app est disponible depuis : http://localhost:8080

#### 1.1 Configuration de Dockerfile / .dockerignore

Créer un fichier Dockerfile à la racine de `backend` et copier dedans le Dockerfile officiel du site Nodejs : 
```Dockerfile 
# Dockerfile
# Layer 1: Image node:10
FROM node:10

# Layer 2: Création du dossier de l'app
WORKDIR /usr/src/app

# Layer 3: Installer les dépendances.
COPY package*.json ./
RUN npm install

# Layer 4: Copier le contenu de l'app
COPY . .

# Layer 5: Port à écouter par le container
EXPOSE 8080

# Layer 6: Run de la commande node server.js
CMD [ "node", "server.js" ]
```

Créer également un .dockerignore afin d'éviter la copie des node_modules sur le container : 
```dockerfile
# .dockerignore:
node_modules
```
#### 1.2 Monter le container

Build l'image : 
```bash
docker build -t backend-node-image .
```

Puis run le container : 
```bash 
docker run -it --rm --name backend --network my-network -p 8080:8080 -v data:/data/node backend-node-image
```

### 2. Database (MongoDb)

Démarrer un container MongoDb à l'aide de la commande suivante : 
```bash 
docker run --name mongo-database --network my-network -d -p 27017:27017 mongo
```
Il devient donc possible de se connecter à l'instance MongoDb via : mongodb://localhost:27017

Commande utile : 
Stopper et supprimer le container :
```bash
docker stop mongo-database && docker rm mongo-database
```

Pour persister les données stocker dans le container, ajouter un volume avec un user root via le flag -v : 
```bash 
docker run --name mongo-database --network my-network -d -p 27017:27017 -v data:/data/db -e MONGO_INITDB_ROOT_USERNAME=user -e MONGO_INITDB_ROOT_PASSWORD=pass mongo
```

### 4. Frontend (React.js)

Créer une application React.js depuis : 
```bash
npx create-react-app frontend
```

Depuis la racine de l'app (`frontend`), ajouter un Dockerfile : 
```dockerfile
# Dockerfile
# Layer 1: Image node 
FROM node:17-alpine 

# Layer 2: Définir le dossier route du container
WORKDIR /app 

# Layer 3: copy les package.json dans le dossier du container
COPY package.json . 

# Layer 4: Installation des dépendances
RUN npm install 

# Layer 5: Copier tous les fichiers depuis le dossier actuel vers le container
COPY . . 

# Layer 6: Définition du port
EXPOSE 3000 

# Layer 7: Run npm start
CMD [“npm”, “start”] 
```

Ajouter un .dockerignore : 
```dockerfile
node_modules
Dockerfile
.git
.gitignore
.dockerignore
```

Depuis le dossier `frontend` build l'image du projet : 
```bash 
docker build -t frontend-react-image .
```

Run le container : 
```bash
docker run -it --rm --name frontend --network my-network -v "/${PWD}:/app" -v /app/node_modules -p 3001:3000 -e CHOKIDAR_USEPOLLING=true frontend-react-image
```

### Network 

Si le network n'a pas été créer en amont et initialiser lors des commandes run des containers, il est nécessaire de suivre cette documentation.

Il est possible de préciser le network dans les commandes de run des container (exemple) :
```bash
docker run -it --rm --name backend --network my-network -p 8080:8080 -v data:/data/node backend-node-image
```

Il est également possible de connecter les container manuellement au même network.
Il faut s'assurer d'avoir créer le network et que les containers soit run : 
```bash
docker network connect my-network frontend
docker network connect my-network backend
docker network connect my-network mongo-database
```

## Docker compose

### 1. LAMP

#### 1.1 Configuration de l'environnement 

Avant tout, créer le dossier de l'application ainsi qu'un fichier php index dans un sous-dossier : 
```bash 
mkdir -p lamp-docker/DocumentRoot
echo "<?php phpinfo(); ?>" > lamp-docker/DocumentRoot/index.php
```

#### 1.2 Php Apache

Ajouter un nouveau dossier dans l'app `lamp-docker` et nommé le php-apache.
Dans ce dossier, ajouter un Dockerfile : 
```dockerfile
FROM php:7.2.1-apache
MAINTAINER egidio docile
RUN docker-php-ext-install pdo pdo_mysql mysqli
```

Ensuite il faut créer un fichier docker-compose.yml à la racine du projet (`lamp-docker`) : 
```dockerfile
version: '3'
services:
    php-apache:
        build:
            context: ./php-apache
        ports:
            -  80:80
        volumes:
            - ./DocumentRoot:/var/www/html
        links:
            - 'mariadb'
```

#### 1.3 Database

Ajouter le code suivant au fichier docker-compose.yml : 
```dockerfile
mariadb:
    image: mariadb:10.1
    volumes:
        - mariadb:/var/lib/mysql
    environment:
        TZ: "Europe/Rome"
        MYSQL_ALLOW_EMPTY_PASSWORD: "no"
        MYSQL_ROOT_PASSWORD: "rootpwd"
        MYSQL_USER: 'testuser'
        MYSQL_PASSWORD: 'testpassword'
        MYSQL_DATABASE: 'testdb'
```

#### 1.4 PhpMyAdmin

Ajouter la partie PhpMyAdmin au docker-compose.yml : 
```dockerfile 
phpmyadmin:
    image: phpmyadmin
    restart: always
    ports:
      - 8080:80
    environment:
      - PMA_ARBITRARY=1
```


#### 1.5 Volumes

Ajouter la partie des volumes au docker-compose.yml : 
```dockerfile
volumes:
    mariadb:
```

Le fichier final doit ressembler à ça : 
```dockerfile
version: '3'
services:
    php-apache:
        image: php:7.2.1-apache
        ports:
            - 80:80
        volumes:
            - ./DocumentRoot:/var/www/html:z
        links:
            - 'mariadb'

    mariadb:
        image: mariadb:10.1
        volumes:
            - mariadb:/var/lib/mysql
        environment:
            TZ: "Europe/Rome"
            MYSQL_ALLOW_EMPTY_PASSWORD: "no"
            MYSQL_ROOT_PASSWORD: "rootpwd"
            MYSQL_USER: 'testuser'
            MYSQL_PASSWORD: 'testpassword'
            MYSQL_DATABASE: 'testdb'

    phpmyadmin:
        image: phpmyadmin
        restart: always
        ports:
          - 8080:80
        environment:
          - PMA_ARBITRARY=1
        links:
          - 'mariadb'
volumes:
    mariadb:

```

Il ne reste qu'à build l'environnement : 
```bash 
docker-compose up
```

Une fois l'application build, accèder à l'adresse suivante pour voir le fichier index.php : 

http://localhost

### 2. MERN
