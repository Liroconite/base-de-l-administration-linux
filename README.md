# base-de-l-administration-linux

## Etape 1

### Pour créer un groupe

On fait la commande:
 ```
 sudo groupadd -g GID NomDuGroupe
```

### Pour créer un utilisateur

On fait la commande: 
```
sudo adduser NomDeL'utilisateur
```

### Pour ajouter un utilisateur dans un groupe

On fait la commande: 
```
sudo usermod -a -G NomDuGroupe NomDeL'utilisateur
```

### Pour vérifier si tout est bon

On fait 
```
su NomDeL'utilisateur
``` 
puis 
```
groups
```
 pour voir s'il est dans le groupe demandé.

## Etape 2

On fait 
```
sudo visudo
``` 
pour aller dans le fichier sudoers. Puis on ajoute
``` 
%admin ALL=(ALL) ALL 
```
pour que tous les membres du groupe admin puissent exécuter les commandes d’administration du système.

## Etape 3

Dans un premier temp nous allons générer une clé SSH depuis votre vm qui va permettre par la suite de ce connecter à d'autre Vm sans besoin de renseigner le mot de passe.

Commande pour générer une clé SSH de type RSA en 2048 bits pour votre compte utilisateur:
```
ssh-keygen -t rsa -b 2048
```

Ensuite, après avoir générer la clé SSH de type RSA en 2048 bits, la prochaine étape, c'est de copier la clé public dans la vm de l'autre utilisateur, pour cela nous allons faire la commande suivante:

Copier la clé dans l'autre VM, ici nous allons copier la clé dans la vm de bob:
```
ssh-copy-id bob@adresse_IP
```

Cela va permettre de ce connecter à la VM de bob sans mot de passe.

Pour verifier si ça fonctionne il faut faire la commande suivante:
```
ssh bob@adresse_IP
```

## Etape 4

Installation de Wordpress:

###Installer Apache2:

```
sudo apt install Apache2
```

Une fois l'installation terminée, mettez à jour les paramètres de votre pare-feu afin d'autoriser le trafic HTTP.

```
sudo ufw allow in "Apache"
```
###Installer Mysql:

Wordpress dépend fortement de la présence d'une base de données, donc nous allons devoir installer Mysql:

```
sudo apt install mysql-server
```

Ensuite taper la commande suivante permettant de sécurisé votre base de données:

```
sudo mysql_secure_installation
```

Appuyer sur Yes pour tout acceptés.

###Installation de PHP

Et maintenant que votre serveur web Apache fonctionne et que la base de données est prête. Le dernier paquetage restant à installer est PHP. Le système de gestion de contenu WordPress est basé sur PHP, il a donc besoin de PHP pour pouvoir fonctionner sur votre serveur. 

```
sudo apt install php libapache2-mod-php php-mysql
```

####Optionnel
voir la version de PHP:

```
php -v
```

Maintenant que vous avez installé PHP, il est important d'installer ces extensions supplémentaires qui sont nécessaires/utiles pour que WordPress fonctionne de manière optimale : 

```
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
```

après avoir installer tout ce qu'il faut, restart Apache

```
sudo systemctl restart Apache2
```

###  Créer un fichier d'hôte virtuel

```
sudo mkdir /var/www/Welcome.com
```

Ensuite, attribuez à l'utilisateur actuel la propriété de ce répertoire : 

```
sudo chown -R $USER:$USER /var/www/Welcome.com
```

Maintenant, ouvrez un nouveau fichier de configuration, en exécutant la commande suivante : 

```
sudo nano /etc/apache2/sites-available/Welcome.com.conf
```

Cela ouvrira l'éditeur de texte intégré dans la ligne de commande. Continuez à remplir ce fichier avec la configuration suivante :

```
<VirtualHost *:80>
ServerAdmin admin@Welcome.com

ServerName Welcome.com
ServerAlias www.Welcome.com

DocumentRoot /var/www/Welcome
<Directory /var/www/Welcome>
	Options FollowSymLinks
	AllowOverride All
	Require all granted
</Directory>

ErrorLog ${APACHE_LOG_DIR}/Welcome_error.log
CustomLog ${APACHE_LOG_DIR}/Welcome_access.log combined
</VirtualHost>
```

Avant de pouvoir activer cette configuration d'hôte virtuel, nous devons désactiver celle qui est en place par défaut et qui entraîne l'apparition de la page de bienvenue d'Apache : 

```
sudo a2dissite 000-default 
```

Et maintenant que votre fichier de configuration est prêt, vous pouvez exécuter la commande a2ensite pour activer l'utilisation de cette configuration d'hôte virtuel : 

```
sudo a2ensite Welcome.com
```

Ensuite, rechargez Apache pour que ces changements prennent effet : 

```
systemctl reload apache2
```

###Créer une base de données MySQL et un utilisateur pour WordPress

Dans un premier temp, nous aller rentrer dans mysql:

```
sudo mysql -u root -p
```

Maintenant, afin de créer une nouvelle base de données pour WordPress, exécutez la commande suivante: 

```
CREATE DATABASE broute DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

La base de données a maintenant été créée mais nous devons également permettre à WordPress d'y accéder. Et pour que cela soit possible, vous allez devoir créer un utilisateur de la base de données en exécutant la commande suivante : 

```
CREATE DATABASE broute DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

Remplacer le nom d'utilisateur par quelque chose que vous souhaitez utiliser comme utilisateur de la base de données pour WordPress. L'exécution de la commande ci-dessus crée seulement l'utilisateur mais ne lui donne pas les privilèges nécessaires. Afin de lui donner un accès complet à la base de données de WordPress, exécutez la commande suivante. 

```
GRANT ALL PRIVILEGES ON broute.* to broute_user@'localhost';
```

Et maintenant, vous pouvez sauvegarder toutes ces modifications apportées aux permissions et quitter entièrement MySQL. Pour ce faire, exécutez successivement les deux commandes suivantes : 

```
FLUSH PRIVILEGES;
exit
```

###Installation de WordPress

Exécutez-les successivement pour installer la dernière version stable de WordPress: 

```
wget -O /tmp/wordpress.tar.gz https://wordpress.org/latest.tar.gz
sudo tar -xzvf /tmp/wordpress.tar.gz -C /var/www/Welcome.com
sudo chown -R www-data.www-data /var/www/Welcome.com
```

pour voir si wordpress fonctionne vien faire la commande suivante:

```
adresse_IP/wordpress
```
