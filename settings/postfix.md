# Отправку решил настроить через postfix и smtp yandex

Для этого устанавливаем `postfix`
```shell
sudo apt update && sudo apt install postfix -y
```
- При установке нужно выбрать internet site и после ввести доменное имя она будет автоматически подставляться после `@`

**После установки ошибка сразу уходит, но нужно еще настроить отправку писем**

- Далее после установки идем в конфиг postfix
```shell
sudo cp /etc/postfix/main.cf /etc/postfix/main.cf.default #Делаем копию
sudo nano /etc/postfix/main.cf
```
В нем добавляем или изменяем строки
```shell
smtp_sasl_auth_enable = yes # Включает механизм SASL (Simple Authentication and Security Layer) аутентификации для исходящей почты через SMTP.
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd # Указывает файл, где хранятся SASL учетные данные в формате хеша. !! Нужно создать этот файл если его нет
smtp_sasl_security_options = noanonymous # Запрещает анонимную аутентификацию, требуя, чтобы все пользователи идентифицировали себя.
smtp_sasl_tls_security_options = noanonymous # То же, что и smtp_sasl_security_options, но применяется, когда используется TLS (Transport Layer Security).

relayhost = [smtp.yandex.ru]:587 # Определяет внешний SMTP-сервер, который будет использоваться для пересылки исходящей почты

inet_interfaces = loopback-only # Ограничивает все сетевые интерфейсы почтового сервера только loopback интерфейсом, что делает его недоступным с других машин.
inet_protocols = ipv4
masquerade_domains = everylife.ru # Маскирует домен отправителя, заменяя локальные домены на everylife.ru в адресе отправителя.
sender_canonical_maps = hash:/etc/postfix/sender_canonical # Указывает файл для канонической перезаписи адресов отправителей, что позволяет изменять исходящие адреса электронной почты. !! Нужно создать этот файл если его нет
```
После создаем файлы sasl_passwd и sender_canonical
> Не уверен насчет надобности этого файла sender_canonical, так как отправки идет через внешний сервис, ну потом нужно будет провести экскремент отключить этот файл

```shell
nano /etc/postfix/sender_canonical
```
В этом файле нужно емайл который нужно подменить и емайл на которой поменяем
```shell
userOld@exemple.com userNew@exemple
user2Old@exemple.com user2New@exemple
```

По умолчанию отправка идет от пользователя на котором работает nginx, то есть если пользователь www-data то будет отправитель www-data@exemple.com, `Но могу ошибаться, не совсем в этом разобрался`

```shell
nano /etc/postfix/sasl_passwd
```

В этом файле прописываем
```shell
[smtp.yandex.ru]:587 ваш_логин:ваш_пароль
```
* ваш_логин - это адрес почты. которая зарегистрирована в этом внешнем сервисе
* ваш_пароль - Это пароль приложений созданный для этого ящика

После всех манипуляция нужно создать хэшированные файлы чтобы Postfix мог эффективно получать доступ к учетным данным

```shell
sudo postmap /etc/postfix/sender_canonical
sudo postmap /etc/postfix/sasl_passwd
```
И перезагрузить сервис postfix
```shell
sudo systemctl restart postfix
```
Перед тем как тестировать отправки нужно убедиться что у домена правильно настроены записи `MX SPF DKIM`

Для теста нужно установить `mailutils`
```shell
sudo apt install mailutils -y
```
После можно пробовать
```shell
echo "Текст письма" | mail -s "Тема письма" -a "From: sender email" [email protected]
```
В этом примере:
* echo "Текст письма" генерирует тело письма.
*  | (pipe) перенаправляет вывод команды echo в команду mail.
*   -s "Тема письма" задает тему письма.
* From: sender email - емайл от которого будет отправлено письмо
*  [email protected] указывает адрес получателя.

При использовании этого способа проверки письмо не отправилось и появилась ошибка в логах
>SMTPUTF8 is required, but was not offered by host smtp.yandex.ru[77.88.21.158]

Она означает, что сервер smtp.yandex.ru не предложил поддержку SMTPUTF8, хотя она требуется для отправки вашего сообщения.
Добавляем в конфиг такую запись
```shell
smtputf8_enable = no
```
Этот параметр отключает поддержку SMTPUTF8, то есть отправка кириллических символов станет доступна
