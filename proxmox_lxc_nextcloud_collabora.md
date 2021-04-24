
# Proxmox

# Update et installation LAMP, APCu, Redis

```
sudo apt update && sudo apt upgrade
```

## LAMP
```
sudo apt install lamp-server^
```
### Meta-packages
L'utilisation du ^ dans le nom du paquet est importante car elle permet d'installer un ensemble de paquets courament instalés ensemble.
Cela permet d'installer Apache, MySQL et PHP avec les modules nécessaires.

* Pour mysql, ne pas oublier 
```
sudo mysql_secure_installation
```

## APCU et Redis
```
sudo apt install php-apcu redis-server php-redis
```

## Paquets nécessaire à NextCloud
```
sudo apt install php-zip php-dompdf php-xml php-mbstring php-gd php-curl unzip php-intl php-bcmath php-gmp php-imagick imagemagick
```

## Modules Apache
```
sudo a2enmod rewrite headers env dir mime
```

## Relancer pour vérifier
```
sudo service apache2 restart
```


# SSL
* Tutoriel suivi : [https://doc.ubuntu-fr.org/tutoriel/securiser_apache2_avec_ssl]

## Domaine test : cloud.mon-domaine.fr

## renouvellement @todo
```
sudo crontab -e
```



# Nextcloud
Pour des raisons de simplicité, Nextcloud sera installé directement dans /var/www/html. Attention à la version, ici la dernière était la 21.0.1.
```
cd /var/www
sudo wget https://download.nextcloud.com/server/releases/nextcloud-21.0.1.zip
sudo unzip nextcloud-21.0.1.zip
sudo rm html
sudo mv nextcloud html
```
NB : Le répertoire html peux bien évidement être supprimé.

## Droits apache sur le nouveau répertoire
```
sudo chown -R www-data:www-data /var/www/html
```

## Base mysql
```
sudo mysql -u root -p
```

```SQL
CREATE DATABASE nextcloud;
CREATE USER 'ncuser'@'localhost' IDENTIFIED BY 'ncpassword';
GRANT ALL PRIVILEGES ON nextcloud . * TO 'ncuser'@'localhost';
```

## Data dir
Le déplacement du répertoire sera effectué plus tard.

## Relancer apache
```
sudo systemctl restart apache2
```

## Installation
Aller avec votre navigateur sur l'url de votre installation : http://cloud.mon-domaine.fr/ et suivez les instructions.

Attention, un répertoire DATA dans le webroot n'est pas vraiment conseillé sauf si vous avez configuré votre .htaccess en conséquences. Le mieux est de le créer en dehors.
Exemple :
```
sudo mkdir /nextcloud-data
sudo chown -R www-data:www-data /nextcloud-data
```

Utiliser ce répertoire lors de l'installation.

## .htaccess
@vérifier
Si soucis accès :

```
sudo vim /etc/apache2/sites-available/000-default.conf
```

### Modifier (AllowOverride) :
```Apache
<Directory /var/www/html>
        Options -Indexes +FollowSymLinks +MultiViews
        AllowOverride None
        Require all granted
</Directory>
```
En :
```Apache
<Directory /var/www/html>
        Options -Indexes +FollowSymLinks +MultiViews
        AllowOverride All
        Require all granted
</Directory>
```

### Relancer
```
sudo service apache2 restart
```


# Activer le cache 'Redis'

```
sudo vim /etc/redis/redis.conf
```

Modifier :
```
port 6379
```
En :
```
port 0
```

Décommenter 
```
unixsocket /var/run/redis/redis.sock
unixsocketperm 700
```

Et modifier la permission du socket :
```
unixsocketperm 770
```

```
sudo systemctl enable redis-server
sudo systemctl restart apache2
```


## Ajout du cache dans la conf. netxcloud
```
sudo vim /var/www/html/config/config.php
```

```php
'memcache.local' => '\\OC\\Memcache\\Redis',
'memcache.locking' => '\\OC\\Memcache\\Redis',
'filelocking.enabled' => 'true',
'redis' => 
array (
  'host' => '/var/run/redis/redis-server.sock',
  'port' => 0,
  'timeout' => 0.0,
),
```

```
systemctl restart redis-server
```


## Pretty-links
On peut également supprimer l'index.php qui est dans chaque url :
```
sudo vim /var/www/html/config/config.php
```

Ajouter 
```PHP
        'htaccess.RewriteBase' => '/', 
```
Modifier en fonction de votre installation, ici directement dans /. Si vous avez installé dans un répertoire et accédez à nextcloud via : mon-domaine/repertoire, vous devez mettre '/repertoire').

### Mettre à jour le .htaccess
Allez dans le répertoire d'installation
```
cd /var/www/html
sudo -u www-data php occ maintenance:update:htaccess
```

## Upload
Modifier le 'php.ini'.
```
sudo vi /etc/php/7.4/apache2/php.ini
```

Choisissez votre valeur, ici 2Go :
```
upload_max_filesize = 2048M
post_max_size = 2058M
```

Relancer apache
```
systemctl restart apache2
```

Aller ensuite dans l'administration Nextcloud/Sytème et vérifiez que l'upload size correspond à ce que vous avez mis.


# Collabora 
Suivi du tuto officiel : [https://www.collaboraoffice.com/code/linux-packages/]
Et ..
Merci Christophe :

## Forcer ssl
```
sudo loolconfig set ssl.enable true
sudo loolconfig set ssl.termination true
```

## Ajouter un password
```
sudo loolconfig set-admin-password
```

## Ajout du domaine à la white-list
```
loolconfig set storage.wopi.host cloud.mon-domaine.fr
```

## Utiliser les certificats ssl générés pour le domaine
```
loolconfig set ssl.cert_file_path /etc/letsencrypt/live/cloud.mon-domaine.fr/cert.pem
loolconfig set ssl.key_file_path  /etc/letsencrypt/live/cloud.mon-domaine.fr/privkey.pem
loolconfig set ssl.ca_file_path /etc/letsencrypt/live/cloud.mon-domaine.fr/chain.pem
```

Et surtout !
```
chgrp -R ssl-cert /etc/letsencrypt
chmod -R g=rX /etc/letsencrypt
usermod -a -G ssl-cert lool
```

## On relance et on vérifie
```
sudo systemctl restart loolwsd
sudo systemctl status loolwsd
```

Si besoin, firewall :
```
ufw allow in to any port 9980
```

## Il suffit d'ajouter l'url de collabora dans la configuration de Nextcloud.

# Sources
* https://bayton.org/docs/nextcloud/installing-nextcloud-on-ubuntu-16-04-lts-with-redis-apcu-ssl-apache/
* https://www.collaboraoffice.com/code/linux-packages/
* https://doc.ubuntu-fr.org/tutoriel/securiser_apache2_avec_ssl
