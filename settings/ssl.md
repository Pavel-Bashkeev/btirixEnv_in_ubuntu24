# Установка Certbot для LetsEncrypt

Используемые пакеты:
* certbot 
* python3-certbot-nginx

> Пакеты certbot и python3-certbot-nginx используются для автоматизации процесса получения и обновления SSL/TLS сертификатов от Let's Encrypt. Они особенно полезны для веб-серверов, использующих Nginx, поскольку позволяют легко настроить HTTPS.

Если они не установлены

1) Установка пакетов:
```shell
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

2) Получение нового сертификата и настройка Nginx:
```shell
sudo certbot --nginx
```

3) Автоматическое обновление сертификатов:\
Certbot автоматически добавит запись в crontab для обновления сертификатов

```shell
nano /et/crontab 
```
В открывшемся файле добавляем
```shell
0 0 * * * /usr/bin/certbot renew --quiet
```
>Каждый день в 00:00 certbot будет проверять сертификаты сайтов и обновлять по необходимости.