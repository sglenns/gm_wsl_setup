# WSL setup

This was created to document the setting up of a WSL from the initial setup.
The following are assumed already done and set up:

-   Installed and configured WSL2
-   Using an Ubuntu distribution

## WSL DNS management

```sh
sudo nano /etc/wsl.conf
```

copy the following and save the file

```sh
[network]
generateResolvConf=false
```

Then run this

```sh
sudo rm /etc/resolv.conf
sudo nano /etc/resolv.conf
```

copy the following and save the file

```sh
nameserver 8.8.8.8
```

And shutdown wsl

```cmd
wsl --shutdown
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
sudo chown -R www-data:www-data /var/www
sudo chmod -R 775 /var/www
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
  ServerAlias *.local #wildcard catch all
  VirtualDocumentRoot /var/www/%0/public
</VirtualHost>
```

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

## Starting script

```sh
nano $HOME/doinit
```

copy the following and save the file

```sh
git -C ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k pull && sudo apt update && sudo apt upgrade -y --allow-downgrades && sudo apt autoremove && sudo apt autoclean && sudo npm -g update && composer selfupdate && composer global update && sudo service apache2 restart
```

give file execution permissions for current user

```sh
chmod u+x $HOME/doinit
```

## MySQL

Install repository from https://dev.mysql.com/downloads/repo/apt/

```sh
sudo dpkg -i mysql-apt-config_**
sudo apt install mysql-server
sudo service mysql stop
sudo usermod -d /var/lib/mysql/ mysql
sudo service mysql start
sudo mysql_secure_installation
```
