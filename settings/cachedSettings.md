# Настройка memcached для данных redis для сессий

Используемые пакеты:
* php-redis php-memcache
* redis
* memcached

1) Установка пакетов: (если не установлены)

```shell
sudo apt install memcached php-redis php-memcache redis
```

## memcached
1) После разрешаем автозапуск и запускаем сервис кэширования:
```shell
sudo systemctl enable memcached
```

2) Перезапускаем php-fpm: в моем случае это версия `php8.3-fpm`
```shell
systemctl restart php8.3-fpm
```
После идем в `/bitrix/.settings.php`, В CMS Bitrix с версией ядра после 14.0, берутся настройки отсюда до 14.0 в dbconn 

* `/bitrix/.settings.php` добавляем в array
```php
'cache' => array(
       'value' => array (
          'type' => 'memcache',
          'memcache' => array(
              'host' => 'localhost',
              'port' => '11211'
          ),
          'sid' => $_SERVER["DOCUMENT_ROOT"]."#01"
       ),
    ),
```
* `/bitrix/php_interface/dbconn.php`

```shell
define("BX_CACHE_TYPE", "memcache");
define("BX_CACHE_SID", $_SERVER["DOCUMENT_ROOT"]."#01");
define("BX_MEMCACHE_HOST", "localhost");
define("BX_MEMCACHE_PORT", "11211");
```

**Убедиться что порт 11211 открыт**

## redis

1) В конфиге прописываем/меняем
```shell
maxmemory 256mb
maxmemory-policy allkeys-lru
```
2) Перезапускаем redis

```shell
sudo systemctl restart redis.service
```
3) Добавляем в `/bitrix/.settings.php` в array

```php
'session' => array(
     'handler' => 'redis',
    'host' => 'localhost', // Адрес сервера Redis
    'port' => 6379,       // Порт сервера Redis
 )
```

