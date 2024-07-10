## Обновление списка пакетов и установка необходимых зависимостей
```shell
sudo apt update
sudo apt upgrade
sudo apt install software-properties-common
```

## Добавление репозиториев и установка Apache2
```shell
sudo add-apt-repository ppa:ondrej/apache2
sudo apt update
sudo apt install apache2
```

## Установка PHP и необходимых модулей
```shell
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php php-cli php-fpm php-curl php-gd php-intl php-mbstring php-xml php-xmlrpc php-soap php-zip php-mcrypt php-json php-bcmath php-opcache php-redis php-memcache php-geop
```

## Установка NGINX если нужно использовать сжатие через brotli то нужно сперва добавить репозиторий `sudo add-apt-repository ppa:eilander/nginx`

```shell
sudo add-apt-repository ppa:eilander/nginx
sudo update
sudo apt install nginx libnginx-mod-http-brotli
```

## Установка MariaDB
```shell
sudo apt install mariadb-server mariadb-client mariadb-common
```

## Установка Node.js, nvm и npm для push сервера
```shell
sudo apt install -y nodejs
sudo apt install npm
wget -q -O- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash # устанавливаем nvm
```
После установки получаем npm версии 9.2.0 и nodejs 18.19.1
Для обновления команды npm
```shell
sudo npm i -g npm@latest # обновляем npm до последней версии
```
Для обновления nodejs используем команды
```shell
. ~/.bashrc
nvm --version 
nvm install node
```

## Установка Redis
```shell
sudo apt install redis
```
## Вспомогательное ПО которое может включать утилиты типа Git, Composer и т.д.
```shell
sudo apt install git composer htop postfix acl fail2ban certbot python3-certbot-nginx
```

--- 

## Перед настройкой сервера добавим пользователя bitrix
```shell
useradd -m bitrix
passwd bitrix
```
---
