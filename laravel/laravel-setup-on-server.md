
## Setup Laravel (PHP, MySQL, Node.js) on server with a bash script

Setting up the app on the server is the next important task. In my years of experience, I've a lot of cases when I needed to deploy my Laravel app to the server. It can be AWS EC2, DigitalOcean, Linode or any other platform. Every time I looked for the commands to install this or that software part, which took some time.
For that reason, I've made some ready-made stuff to do that without doing repetitive problem-solving.

In this article, I will share a single bash script that will deploy not only a Laravel app on the server, but install and setup some other related technologies for you as well.
I have tested the script using on AWS EC2, but can use it elsewhere.

If you want to check out the ready script now, here's the [⭐ GitHub Gist](https://gist.github.com/boolfalse/87969129638e77db031a40b49ddf7aea) for that.

<img src="https://i.imgur.com/EjCWxvb.png" style="width: 100%;">

_Setup Laravel with PHP, MySQL, Node.js and ElasticSearch on a server (Ubuntu 22.04)_

### STEPS:

Below are the steps that will be implemented during running the custom bash script:

- Install Node.js (v20.x in the example).
- Install Apache.
- Install MySQL (v8.* in the example).
- Setup MySQL user, role, privileges, database.
- Install PHP (v8.2 in the example).
- Install Composer.
- Setup the project.
- Setup server.
- Setup project's files-directories permissions/ownerships.
- Setup ElasticSearch (v7.x in the example).

### HOW-TO-USE INSTRUCTIONS:

For using a ready script for your own, you need to use these instructions:

- Login to your server (instance):
```shell
ssh -i ~/.ssh/private-key.pem ubuntu@<SERVER_IPv4>
```

- Create or place the setup.sh file with the content provided somewhere in your server. It's recommended to have that in the home directory:
```shell
cd ~
```

- Allow **_setup.sh_** for execution permission:
```shell
chmod +x setup.sh
```

- Run shell script using project's remote repo (GitHub repo is in our case) URL as a parameter:
```shell
./setup.sh "git@github.com:<USERNAME>/<REPO>.git"
```

- **That's it**. At this point you can check the public URL of the instance after the script is done.

### SNIPPETS:

Now I will write all the snippets for each step, so in case you want to use them separately:

- **Install Node.js 20.x**

```shell
echo "1. *** INSTALL NODE 20.x"
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt update
sudo apt-get install -y nodejs
```

- **Install Apache**

```shell
echo "2. *** INSTALL APACHE"
sudo apt update
sudo apt install -y apache2
```

- **Install and Setup MySQL**

```shell
echo "3. *** INSTALL AND SETUP MYSQL"
sudo apt update
sudo apt install -y mysql-server
MYSQL_PASSWORD=$(tr -dc 'A-Za-z0-9!"#$%&'\''()*+,-./:;<=>?@[\]^_`{|}~' </dev/urandom | head -c 13  ; echo)
echo "Please use the generated password below as a MySQL password !!!"
echo "***************************************************************"
echo "***************************************************************"
echo "********************                       ********************"
echo "********************     ${MYSQL_PASSWORD}     ********************"
echo "********************                       ********************"
echo "***************************************************************"
echo "***************************************************************"
echo "Please use the generated password above as a MySQL password !!!"

# No - (Would you like to setup VALIDATE PASSWORD component?)
# $MYSQL_PASSWORD
# $MYSQL_PASSWORD
# y - Remove anonymous users?
# No - Disallow root login remotely?
# No - Remove test database and access to it?
# y - Reload privilege tables now?

sudo mysql_secure_installation <<EOF
No
$MYSQL_PASSWORD
$MYSQL_PASSWORD
y
No
y
y
EOF
```

- **Working with MySQL**

```shell
echo "4. *** WORKING WITH MYSQL"
touch mysql.sql
echo " " > mysql.sql
sed -i "1i FLUSH PRIVILEGES;" mysql.sql
sed -i "1i GRANT ALL PRIVILEGES ON * . * TO 'project'@'localhost';" mysql.sql
sed -i "1i CREATE USER 'project'@'localhost' IDENTIFIED BY \"${MYSQL_PASSWORD}\";" mysql.sql
sudo mysql < "mysql.sql"
rm mysql.sql
mysql -u project -p$MYSQL_PASSWORD -Bse "CREATE DATABASE project"
```

- **Install PHP 8.2**

```shell
echo "5. *** INSTALL PHP-8.2"
sudo apt update
sudo apt -y install software-properties-common
sudo add-apt-repository ppa:ondrej/php
# [ENTER]
sudo apt update
sudo apt -y install php8.2
sudo apt update
sudo apt install -y php8.2-cli php8.2-json php8.2-common php8.2-mysql php8.2-zip php8.2-gd php8.2-mbstring php8.2-curl php8.2-xml php8.2-bcmath php8.2-soap
sudo apt update
sudo apt install unzip
```

- **Install Composer**

```shell
echo "6. *** INSTALL COMPOSER"
curl -sS https://getcomposer.org/installer -o composer-setup.php
HASH=`curl -sS https://composer.github.io/installer.sig`
php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
rm composer-setup.php
```

- **Setup the Project**

```shell
echo "7. *** SETUP THE PROJECT"
cd /var/www/
sudo mkdir web
sudo chown -R $USER:$USER web/
cd web/
git clone $1 project
cd project/
composer install
cp .env.example .env
sed -i -e "s/DB_DATABASE=laravel/DB_DATABASE=project/g" .env
sed -i -e "s/DB_USERNAME=root/DB_USERNAME=project/g" .env
sed -i -e "s/DB_PASSWORD=/DB_PASSWORD=\"${MYSQL_PASSWORD}\"/g" .env
php artisan key:generate
php artisan storage:link
php artisan route:clear
php artisan config:cache
php artisan optimize
php artisan migrate
php artisan db:seed
```

- **Setup Apache Configs (as known as Virtual Server)**

```shell
echo "8. *** SETUP VIRTUAL SERVER"
touch 000-default.conf
echo "<VirtualHost *:80>
        #ServerName project.site
        DocumentRoot /var/www/web/project/public
        <Directory "/var/www/web/project">
                Options Indexes FollowSymLinks
                AllowOverride all
                Require all granted
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>" > 000-default.conf
sudo chmod 644 000-default.conf
sudo chown root:root 000-default.conf
sudo mv /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default_backup.conf
sudo mv 000-default.conf /etc/apache2/sites-available/
sudo a2enmod rewrite
sudo systemctl restart apache2
```

- **Setup Project files-folders Permissions/Ownerships**

```shell
echo "9. *** SETUP PROJECT FILES-FOLDERS PERMISSIONS/OWNERSHIPS"
# APACHE_USERNAME=$(ps -ef | egrep '(httpd|apache2|apache)' | grep -v `whoami` | grep -v root | head -n1 | awk '{print $1}')
APACHE_USERNAME="www-data"
sudo usermod -a -G $APACHE_USERNAME $USER
cd /var/www/web/project
sudo chown -R $USER:$APACHE_USERNAME storage/ bootstrap/cache/
sudo chgrp -R $APACHE_USERNAME storage bootstrap/cache/
sudo chmod -R ug+rwx storage bootstrap/cache/
git checkout .
```

- **Setup ElasticSearch 7.x**

```shell
echo "10. *** SETUP ELASTICSEARCH 7.x"
cd ~
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install elasticsearch
#sudo systemctl start elasticsearch
#sudo systemctl enable elasticsearch
```

- **All done!**

***

If you liked this article, please feel free to follow me here. 😇

To explore projects working with various modern technologies, you can follow me on [**GitHub**](https://github.com/boolfalse), where I actively publicize much of my work.

For more info you can visit my website [**boolfalse.com**](https://boolfalse.com/)

Thank you !!!

