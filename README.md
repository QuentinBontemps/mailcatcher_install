# Installation de mailcatcher sur debian 9 (stretch)

Mailcatcher est un outil utilisé lors des phases de développement ou de test d'un projet.

Il va agir comme un SMTP et va récupérer les mails envoyés quel que soit le destinataire pour les stocker, les lister, et afficher leur contenu.

Il suffit de configurer son projet sur le port SMTP du mailcatcher et tous les mails termineront leur course sur votre mailcatcher. 

Plus aucun mail ne sera envoyé au destinataire final alors qu'il n'aurait pas dû...

## Installation des paquets de base 
```bash
sudo apt-get install linux-headers-$(uname -r)
sudo apt-get install build-essential rubygems ruby-dev libsqlite3-dev sqlite3
```

## Installation de mailcatcher
```bash
gem install mailcatcher
```

## Démarrer Mailcatcher
Il suffit de lancer la commande
```bash
mailcatcher
```
Par défaut, les ports sont :
- 1025 pour le SMTP
- 1080 pour l'interface web


L'interface de mailcatcher est désormais disponible sur ```mailcatcher.mondomain.fr:1080```

Si vous souhaitez que le mailcatcher soit accessible sur le port 80, il faut faire un reverse proxy.

## Exemple pour Apache:
### Activation des modules 
Pour utiliser Apache en mode reverse proxy, il faut activer deux modules ```proxy``` et ```proxy_http``` via la commande ```a2enmod```
```bash
sudo a2enmod proxy proxy_http
```

### Création du vhost
```bash
vi /etc/apache2/sites-available/mailcatcher.conf
```
Coller la configuration du vhost ci-dessous, en l'adaptant à vos informations
```apacheconfig
# Mailcatcher config
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass / http://localhost:1080/
    ProxyPassReverse / http://localhost:1080/
    ServerName mailcatcher.mondomaine.fr
    # Protection par htpassword pour qu'il ne soit pas accessible à tous
    <Proxy *>
        Order deny,allow
        Deny from all
        AuthType Basic
        AuthName "MailCatcher"
        AuthUserFile /var/www/common/.htpasswd
        Require valid-user
        Satisfy any
    </Proxy>
</VirtualHost>
```

Activer le vhost via la commande ```a2ensite```
```bash
sudo a2ensite mailcatcher
```

Rechargement de la configuration Apache
```bash
sudo service apache2 reload
```
L'interface de mailcatcher est désormais disponible sur ```mailcatcher.mondomain.fr```

## Exemple configuration projet
### Laravel
Pour configurer le SMTP de mailcatcher dans un projet Laravel, il faut mettre cette configuration dans le fichier ```.env```
```
MAIL_DRIVER=smtp
MAIL_HOST=127.0.0.1
MAIL_PORT=1025
MAIL_USERNAME=
MAIL_PASSWORD=
```

### Synfony
Pour configurer le SMTP de mailcatcher dans un projet Synfony, il faut mettre cette configuration dans le fichier ``` app/config/parameters.yml```
```yaml
mailer_transport: smtp
mailer_host: '127.0.0.1:1025'
mailer_user: null
mailer_password: null
```
