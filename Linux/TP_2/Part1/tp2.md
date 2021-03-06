# TP2 pt. 1 : Gestion de service

## I. Un premier serveur web

### 1. Installation

#### ๐ Installer le serveur Apache
โโโ
[yaniss@web ~]$ sudo dnf install -y httpd

โโโ

#### ๐ Dรฉmarrer le service Apache
โโโ

[yaniss@web ~]$ systemctl enable httpd.service
โโโ
โโโ

[yaniss@web ~]$ systemctl start httpd.service
โโโ
โโโ


[yaniss@web ~]$ systemctl status httpd.service
โ httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor prese>
   Active: active (running) since Wed 2021-09-29 17:50:35 CEST; 2min 24s ago
โโโ
โโโ
   

[yaniss@web ~]$ sudo firewall-cmd --add-port=80/tcp
โโโ
โโโ


[yaniss@web ~]$ sudo ss -alnpt
LISTEN    0         128                      *:80                      *:*        users:(("httpd",pid=863,fd=4),("httpd",pid=862,fd=4),("httpd",pid=861,fd=4),("httpd",pid=836,fd=4))
โโโ

Pour tester rapidement que mon mon serveur tourne, avec la commande curl localhost:80 j'obtiens bien le contenu de ma page html. Depuis ma propre machine je rentre l'ip de ma machine suivi du port 80 et j'obtiens aussi le contenu de ma page html.


### 2. Avancer vers la maรฎtrise du service

#### ๐ Le service Apache...

Voici la commande qui permet d'activer le dรฉmarrage automatique d'Apache quand la machine s'allume
[yaniss@web ~]$ systemctl enable httpd.service

Avec la commande suivante je prouve que le service est parametrer pour demarrer au demarage.
โโโ

[yaniss@web ~]$ systemctl is-enabled httpd.service
enabled 
โโโ

J'affiche le contenu du fichier httpd.service.
โโโ

[yaniss@web ~]$ cat /usr/lib/systemd/system/httpd.service
#	[Service]
#	Environment=OPTIONS=-DMY_DEFINE

[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true

[Install]
WantedBy=multi-user.target
โโโ

๐ Dรฉterminer sous quel utilisateur tourne le processus Apache
โโโ

[yaniss@web ~]$ cat /etc/httpd/conf/httpd.conf 
User apache
โโโ

Avec la commade ps -ef je visualise le process apache et l'user avec lequel le process tourne.
โโโ

[yaniss@web ~]$ ps -ef
root        1920       1  0 17:17 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      1921    1920  0 17:17 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      1922    1920  0 17:17 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      1923    1920  0 17:17 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache      1924    1920  0 17:17 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
โโโ

Je vรฉrifie ensuite que tout le contenu est accesible ร? l'utilisateur mentionnรฉ dans le fichier de conf
โโโ

[yaniss@web ~]$ ls -al /var/www/
drwxr-xr-x.  4 root root   33 29 sept. 12:32 .
drwxr-xr-x. 22 root root 4096 29 sept. 12:32 ..
drwxr-xr-x.  2 root root    6 11 juin  17:35 cgi-bin
drwxr-xr-x.  2 root root    6 11 juin  17:35 html
โโโ

On remarque que tout le dossier est executable et lisile pour le groupe apache.


### ๐ Changer l'utilisateur utilisรฉ par Apache

J'ajoute un nouvel user 
โโโ

[yaniss@web ~]$ sudo useradd -d /usr/share/httpd -s /sbin/nologin  myUser

โโโ

Dans le fichier /etc/httpd/conf/httpd.conf je modifie la ligne ou l'user est apache par myUser.
Puis de nouveau avec la commande ps -ef.
โโโ

[yaniss@web ~]$ ps -ef
newUser     1820    1819  0 18:05 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
newUser     1821    1819  0 18:05 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
newUser     1822    1819  0 18:05 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
newUser     1823    1819  0 18:05 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
โโโ

๐ Faites en sorte que Apache tourne sur un autre port

Je modifie la ligne spรฉcifiant sur quel port รฉcoute le serveur en rajoutant 8080 a la place de 80, tout en n'oubliant pas d'ajouter le port 8080 en tcp dans la conf du service firewall.

En relancant le service, on remarque avec la commande SS(pas Schutzstaffel bien sur) que le port 8080 fait tourner bien un service.
โโโ

[yaniss@web ~]$ sudo ss -alnpt
[...]
LISTEN    0         128                      *:8888                    *:*        users:(("httpd",pid=2155,fd=4),("httpd",pid=2154,fd=4),("httpd",pid=2153,fd=4),("httpd",pid=2150,fd=4))
[...]
โโโ

De nouveau, pour tester rapidement que mon mon serveur tourne, avec la commande curl localhost:8080 j'obtiens bien le contenu de ma page html. Depuis ma propre machine je rentre l'ip de ma machine suivi du port 8080 et j'obtiens aussi le contenu de ma page html.

### ๐ Install du serveur Web et de NextCloud sur web.tp2.linux

Voici grace a mon history des commandes les รฉtapes nรฉcessaires a l'installation de NextCloud
โโโ

sudo dnf install epel-release -y
โโโ
โโโ

sudo dnf update -y
โโโ
โโโ

sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm -y
โโโ
โโโ

sudo dnf module enable php:remi-7.4 -y
โโโ
โโโ

sudo dnf install httpd mariadb-server vim wget zip unzip libxml2 openssl php74-php php74-php-ctype       php74-php-curl php74-php-gd php74-php-iconv php74-php-json php74-php-libxml php74-php-mbstring php74-   php-openssl php74-php-posix php74-php-session php74-php-xml php74-php-zip php74-php-zlib php74-php-     pdo php74-php-mysqlnd php74-php-intl php74-php-bcmath php74-php-gmp -y
โโโ
โโโ

sudo mkdir /etc/httpd/sites-available
โโโ
โโโ

sudo ln -s /etc/httpd/sites-available/linux.web /etc/httpd/sites-enabled
โโโ
โโโ

sudo nano /etc/httpd/sites-available/linux.web
โโโ
โโโ

sudo mkdir /etc/httpd/sites-enabled
โโโ
โโโ

sudo ln -s /etc/httpd/sites-available/linux.web
โโโ
โโโ

sudo nano /etc/opt/remi/php74/php.ini
โโโ
โโโ

sudo nano /etc/opt/remi/php74/php.ini
โโโ
โโโ

sudo wget https://download.nextcloud.com/server/releases/nextcloud-21.0.1.zip
โโโ
โโโ

sudo unzip nextcloud-21.0.1.zip
โโโ
โโโ

cd nextcloud/
โโโ
โโโ

sudo cp -Rf * /var/www/sub-domains/linux.web/html/
โโโ
โโโ

sudo chown -Rf apache.apache /var/www/sub-domains/linux.web/html/


โโโ
โโโ

[yaniss@db ~]$ sudo dnf install epel-release -y
โโโ
โโโ

[yaniss@db ~]$ sudo dnf update -y
โโโ
โโโ

[yaniss@db ~]$ sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm -y
โโโ
โโโ

[yaniss@db ~]$ sudo dnf module enable php:remi-7.4 -y
โโโ
โโโ

[yaniss@db ~]$ sudo dnf install httpd mariadb-server vim wget zip unzip libxml2 openssl php74-php php74-php-ctype php74-php-curl php74-php-gd php74-php-iconv php74-php-json php74-php-libxml php74-php-mbstring php74-php-openssl php74-php-posix php74-php-session php74-php-xml php74-php-zip php74-php-zlib php74-php-pdo php74-php-mysqlnd php74-php-intl php74-php-bcmath php74-php-gmp -y
โโโ
โโโ

[yaniss@db ~]$ sudo dnf install -y mariadb-server
โโโ
โโโ

[yaniss@db ~]$ mysql_secure_installation
โโโ
โโโ

[yaniss@db ~]$ sudo systemctl enable mariadb
โโโ
โโโ

[yaniss@db ~]$ sudo systemctl restart mariadb
โโโ
โโโ

[yaniss@db ~]$ ss -alnpt
โโโ
### ๐ Prรฉparation de la base pour NextCloud


#### trouver une commande qui permet de lister tous les utilisateurs de la base de donnรฉes

Avec la comamande SELECT User FROM mysql.user;  

### C. Finaliser l'installation de NextCloud


### ๐ Exploration de la base de donnรฉes

Avec la commande suivante je verifie le nombre de lignes crรฉes.
โโโ

USE nextcloud SELECT FOUND_ROWS();
โโโ

| Machine           | IP            | Masque            | Service                 | Port ouvert       | IP autorisรฉes|
|-------------------|---------------|-------------------|-------------------------|-------------------|--------------|
| web.tp2.linux     | `10.3.0.126` | `255.255.255.0`   | Machine Web             | 80/tcp - 22/tcp   |toutes les ip |
| db.tp2.linux      | `10.3.0.122` | `255.255.255.0`   | Machine hostant la Base de Donnรฉes | 3306/tcp - 22/tcp |`10.3.0.126` |




