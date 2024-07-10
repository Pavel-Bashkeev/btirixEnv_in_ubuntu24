---
* Выполнение агентов на cron
  Ошибка! Не настроен запуск cron_events.php на cron, последний агент отработал больше суток назад.

  {

    Для исправления этой ошибки, пишем в терминале
    ```shell
    crontab -e -u bitrix
    ```
    * Команда `crontab -e -u bitrix` используется для редактирования crontab-файла пользователя `bitrix`.

    В открывшемся файле добавляем
    ```shell
    */10 * * * * /usr/bin/php -f /home/bitrix/www/bitrix/modules/main/tools/cron_events.php
    ```
    **Где : \*/10 \* \* \* \* - означает раз в десять минут.**

  }

---

* Отправка почты
  Ошибка! Не работает\
  {

}

---

* Отправка почтового сообщения больше 64Кб
  Ошибка! Не работает
  {

Эта ошибка уйдет в след за предыдущей

}

---

* Проверка на наличие неотправленных сообщений
  Ошибка! Есть ошибки при отправке системных почтовых сообщений, число неотправленных сообщений: 1
  Определена константа BX_CRONTAB_SUPPORT в /bitrix/php_interface/dbconn.php, при этом должен быть настроен вызов
  агентов на cron.

{

Для решения этой проблемы требуется создать еще один файл cron_events.php в директории bitrix/php_interface

В этом файле пишем такой код

```php
// Установка корневого каталога документов один раз, чтобы избежать повторения
$documentRoot = realpath(dirname(__FILE__)."/../..");


use Bitrix\Main\Loader;

require($documentRoot."/bitrix/modules/main/include/prolog_before.php");
// Подключение модуля 'main'
if (!Loader::includeModule('main')) {
    throw new \Exception('Не удалось подключить модуль "main"');
}

use Bitrix\Main\Application;
use Bitrix\Main\SiteTemplateTable;
use Bitrix\Main\Config\Configuration;
use Bitrix\Main\Mail\Event;
use Bitrix\Sender\MailingManager;

// Установка корневого каталога документов

//Configuration::setValue('exception_handling', ['value' => ['debug' => true]]);

// Определение констант для управления поведением скрипта
define("NO_KEEP_STATISTIC", true);
define("NOT_CHECK_PERMISSIONS", true);
define('BX_CRONTAB_SUPPORT', true);
define('BX_CRONTAB', true);

// Подключение необходимых модулей Bitrix
if (Loader::includeModule('main')) {
    // Игнорирование ограничений времени выполнения и отключения пользователя
    set_time_limit(0);
    ignore_user_abort(true);

    // Выполнение задач агента
    CAgent::CheckAgents();

    // Проверка и отправка почтовых рассылок, если модуль 'sender' доступен
    if (Loader::includeModule('sender')) {
        MailingManager::checkPeriod(false);
        MailingManager::checkSend();
    }

    // Выполнение резервного копирования и финальных действий
    Event::sendImmediate();
    Application::FinalActions();
}

```

После вызываем опять файл corn

```shell
crontab -e -u bitrix
```

И добавляем запись

```shell
 */1 * * * * /usr/bin/php -f /home/bitrix/www/bitrix/php_interface/cron_events.php
```

Проверяем работу

```shell
systemctl status cron
```

* Такой момент нужно обязательно убелиться что в параметрах php установлен параметр `short_open_tag=On`

Обычно он находится в /etc/php/x.x/cli/php.ini для CLI и нужно также установить этот параметр в /etc/php/x.x/fpm/php.ini

}

---

* HTTP авторизация
  Ошибка! Не работает\
  {

> В настройках php в моем случае это /etc/php/8.3/fpm/php.ini\
> Меняем значения опции:\
> **cgi.rfc2616_headers** устанавливаем значение 1

**cgi.rfc2616_headers** - Эта опция указывает PHP отправлять заголовки в соответствии со стандартом RFC 2616. Когда эта
опция включена, PHP будет использовать более строгие правила для HTTP заголовков, что может улучшить совместимость с
некоторыми веб-серверами и клиентами.


> **cgi.fix_pathinfo** устанавливаем значение 0

**cgi.fix_pathinfo** - Эта настройка влияет на то, как PHP обрабатывает URL-запросы к скриптам. Когда установлено в
значение 1, PHP пытается угадать, где находится скрипт и какая часть строки запроса является фактическим путем к
скрипту. Это может создать проблемы безопасности, так как злоумышленники могут попытаться эксплуатировать неправильно
настроенные серверы. Поэтому рекомендуется установить эту опцию в 0, чтобы PHP не интерпретировал данные в URL за
пределами пути к скрипту.

Так как у меня сервер работает в связке nginx + php-fpm то в блоке `location ~ \.php$` прописываем передача
параметра `REMOTE_USER` в `fastcgi_param` значение `$http_authorization`
> fastcgi_param REMOTE_USER $http_authorization;

После в файле `/php_interface/dbconn.php`, добавляем

```php
$remote_user = $_SERVER["REMOTE_USER"] ? $_SERVER["REMOTE_USER"] : $_SERVER["REDIRECT_REMOTE_USER"];
$strTmp = base64_decode(substr($remote_user,6));
if ($strTmp) {
	list($_SERVER['PHP_AUTH_USER'], $_SERVER['PHP_AUTH_PW']) = explode(':', $strTmp); 
}
```

}