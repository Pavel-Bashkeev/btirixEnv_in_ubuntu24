
1) Настройка Apache2:

Нужно изменить порт, который Apache2 использует по умолчанию, чтобы избежать конфликта с NGINX:
* Открываем файл `/etc/apache2/ports.conf` и меняем  `Listen 80` на `Listen 8080`
```shell
nano /etc/apache2/ports.conf
```