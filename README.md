# WSL setup

This was created to document the setting up of a WSL from the initial setup.
The following are assumed already done and set up:

- Installed and configured WSL2
- Using an Ubuntu distribution

## WSL DNS management

```sh
sudo rm /etc/resolv.conf
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
sudo bash -c 'echo "[network]" > /etc/wsl.conf'
sudo bash -c 'echo "generateResolvConf = false" >> /etc/wsl.conf'
sudo chattr +i /etc/resolv.conf
```

And shutdown wsl

```ps
wsl --shutdown
```

## disable IPv6 addressing

### on system

Edit `/etc/sysctl.conf`
```sh
sudo nano /etc/sysctl.conf
```

paste at bottom

```sh
#disable ipv6
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
net.ipv6.conf.lo.disable_ipv6=1
```
make changes effetctive

```sh
sudo sysctl -p
```

### on SSH

edit `/etc/ssh/sshd_config`

```sh
sudo nano /etc/ssh/sshd_config
```

and modify

```sh
AddressFamily inet
```

restart ssh
```sh
/scripts/restartsrv_sshd
```

## setting up repositories and add git, php and node

```sh
sudo add-apt-repository ppa:git-core/ppa -y
sudo add-apt-repository ppa:ondrej/php -y
sudo add-apt-repository ppa:ondrej/apache2 -y
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt update && sudo apt upgrade -y && sudo apt autoremove && sudo apt autoclean
sudo apt install git php nodejs -y
```

## composer

```sh
EXPECTED_CHECKSUM="$(wget -q -O - https://composer.github.io/installer.sig)"
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
ACTUAL_CHECKSUM="$(php -r "echo hash_file('sha384', 'composer-setup.php');")"
if [ "$EXPECTED_CHECKSUM" != "$ACTUAL_CHECKSUM" ]
then
    >&2 echo 'ERROR: Invalid installer checksum'
    rm composer-setup.php
    exit 1
fi
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
rm composer-setup.php
```

## change npm -g folder (to avoid calling `sudo` on `npm -g`)

```sh
mkdir ~/.npm
npm config set prefix '~/.npm'
export PATH="$(npm config get prefix)/bin:$(composer config -g home)/vendor/bin:$PATH"
source ~/.profile
```

## git

```sh
git config --global init.defaultBranch "main"
git config --global user.email "YOUR_EMAIL"
git config --global user.name "YOUR_USERNAME"
git config --global pull.ff only
```

## setting `/var/www/` forlder and user permissions

```sh
sudo apt install acl
sudo chown -R www-data:www-data /var/www
sudo chmod -R 775 /var/www
sudo chmod g+s /var/www
sudo setfacl -d -m g::rwx /var/www
sudo setfacl -d -m o::rx /var/www
sudo usermod -g www-data $USER
```

## apache mod installtion and vhost setting

```sh
sudo a2enmod vhost_alias proxy_http expires headers ssl rewrite
sudo nano /etc/apache2/sites-available/wildcard.local.conf
```

copy the following and save the file

```sh
<Directory "/var/www/*/public">
  Options Indexes FollowSymLinks
  AllowOverride All
  Order allow,deny
  Allow from all
  Require all granted
</Directory>
<VirtualHost *:80>
  UseCanonicalName Off
  ServerAlias *.local
  ServerAlias *
  VirtualDocumentRoot /var/www/%0/public
</VirtualHost>
```

change `"/var/www/*/public"` and `/var/www/%0/public` according to necessities

```sh
sudo a2ensite wildcard.local
sudo service apache2 restart
```

> On the hosts file remember to add IPv6 version of the addresses

## laravel required php libs

```sh
sudo apt install openssl php-common php-curl php-json php-mbstring php-mysql php-xml php-zip
```

## OMZ + powerlevel10k

```sh
sudo apt install zsh -y
 sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
nano ~/.zshrc
```

Set the following and save the file

```sh
ZSH_THEME="powerlevel10k/powerlevel10k"
export PATH="$(npm config get prefix)/bin:$(composer config -g home)/vendor/bin:$PATH"
```

Restart zsh

```sh
exec zsh
```

## MySQL

Install .deb file from https://dev.mysql.com/downloads/repo/apt/

```sh
sudo dpkg -i mysql-apt-config_**
sudo apt update
sudo apt install mysql-server
sudo service mysql stop
sudo usermod -d /var/lib/mysql/ mysql
sudo service mysql start
sudo mysql_secure_installation
```

## Starting script

```sh
nano $HOME/doinit
```

copy the following and save the file

```sh
#!/bin/bash

git -C ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k pull && sudo apt update && sudo apt upgrade -y --allow-downgrades && sudo apt autoremove && sudo apt autoclean&& sudo composer selfupdate && composer global update && sudo service apache2 restart && sudo service php8.0-fpm start  && sudo service php8.0-fpm restart && sudo service mysql restart && npm -g update
```

give file execution permissions for current user

```sh
chmod u+x $HOME/doinit
```

to use it

```sh
$HOME/doinit
```

## Cypress
to use Cypress it might be needed to do as follows
```sh
sudo apt update
sudo apt install -y libgbm-dev
```
