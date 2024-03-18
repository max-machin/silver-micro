# silver-micro
Projet de machine virtuelle dockerisée.   
Hyperviseur de type 2 : VMware  
Distribution : Debian 12

## Installation de base 
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

### Stack LAMP
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
