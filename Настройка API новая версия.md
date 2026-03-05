### Инструкция по установке и работе плагина для управления Почтовым сервером для Корпоративного сервера 2024 ОС Astra Linux 1.7.4
Скачать плагин
##### 1) Список необходимых пакетов<br>
1.1. Что должно быть установлено (пакеты / команды)<br>
На сервере Корпоративный сервер 2024<br>
1.1.1. Обязательные пакеты (ставятся установщиком)<br>
1.1.2. Системные компоненты, которые должны быть в системе<br>
##### 2) Назначение плагина и что делает установщик<br>
##### 3) Подготовка перед запуском<br>
3.1. Подготовить файлы рядом со скриптом<br>
3.2. Проверить ожидаемые пути<br>
3.3. Подготовить доступ к mailserver_api<br>
##### 4) Запуск скрипта<br>
4.1. Установка<br>
4.2. Проверка после установки<br>
4.3. Восстановление после обновления основного ПО (guard)<br>
4.4. Удаление<br>
##### 5. Что именно меняется в системе (для аудита)<br>

[Скачать плагин](https://download.r7-office.ru/plugin/plugin-mail-astra174.zip)
<br>MD5: 74644B456277E21F52D7DD51516BFA74

##### Обновление API<br>
Для обновления API Почтового сервера до версии 1.4:<br>
1. Скопировать архив mailserver_api_1.4.tar в корень / <br>
2. Остановить службу и удалить директорию с предыдущей версией API
```bash
systemctl stop mailserver_api.service
rm -rf /opt/r7-office/mailserver_api/
cd /
```
3. Разархивировать и перезапустить службу API
```bash
tar -xzvf mailserver_api_1.4.tar.gz --selinux -C /
systemctl restart mailserver_api.service
systemctl status mailserver_api.service
```
[Скачать API 1.4](https://download.r7-office.ru/mailserver/mailserver_api_1.4.tar.gz)
<br>MD5: 4D13ABE87CAB42E433D36556CBD4AC2C

##### 1) Список необходимых пакетов<br>
1.1. Что должно быть установлено (пакеты / команды)<br>
На сервере Корпоративный сервер 2024<br>
>[!Warning]
>Пакеты находятся вместе с плагином. При оффлайн установке ставятся из pkg/

1.1.1. Обязательные пакеты (ставятся установщиком)<br>
Эти пакеты нужны для:<br>
создания venv и установки Python-зависимостей, работы установщика (скачивание/проверки), создания BasicAuth (htpasswd) для /apimail.<br>
Установить заранее (или доверить установщику):<br>
```bash
apt update
apt install apache2-utils
apt install curl ca-certificates
apt install perl
apt install python3 python3-venv python3-dev python3-pip python3-setuptools python3-wheel
apt install build-essential
apt install libffi-dev libssl-dev
```
1.1.2. Системные компоненты, которые должны быть в системе<br>
Обычно уже присутствуют на сервере Р7, но без них работа невозможна:<br>
```cmd
nginx — проксирование /admin/apimail и /admin/mail-manager
systemd — управление сервисом r7-mail-manager
iproute2 — утилиты ip/ss (используются установщиком при выборе свободного порта)
```
На ОС дополнительно может устанавливить/обновить ряд пакетов. Среди них встречаются:<br>
```cmd
сборочные и dev-компоненты: g++, dpkg-dev, patch, libexpat1-dev, libpython3-dev, libpython3.X-dev
python-модули из репозитория: python3-distutils, python3-lib2to3, python3-entrypoints, python-pip-whl
криптобиблиотеки для Python: python3-cryptography, python3-cffi-backend, python3-asn1crypto
venv/dev для конкретной версии Python (например): python3.7-venv, python3.7-dev
```
Скрипт активно:
```cmd
патчит index.html
модифицирует nginx-конфиг
создает systemd-unit
```
##### 2) Назначение плагина и что делает установщик<br>
Установщик выполняет установку/обновление и при необходимости удаление:<br>
```cmd
Разворачивает backend в /opt/r7-mail-manager (venv + app.py) и лог в /var/log/r7-mail-manager.log
Ставит systemd unit r7-mail-manager.service (из отдельного файла рядом с установщиком)
Копирует JS в /var/www/r7-office/admin/assets/ и добавляет <script> в /var/www/r7-office/admin/index.html
Патчит nginx-конфиг админки /etc/nginx/sites-available/admin (добавляет блоки /apimail и /mail-manager/ между маркерами)
Создаёт конфиг с секретами только на сервере: /etc/r7-mail-manager.env (права 0600)
Создаёт резервную копию важных файлов в /opt/r7-mail-manager/backup/ (для восстановления после обновлений основного ПО)
```
##### 3) Подготовка перед запуском<br>
3.1. Подготовить файлы рядом со скриптом<br>
В одной папке должны лежать:
```bash
r7-mail-manager-install.sh
r7-mail-manager.service
ensure-admin-assets.sh
app.py
inject-manager.js
inject-mail-tab.js
pkg/ -
```
> inject-manager.js копируется только если его ещё нет и используется всеми плагинами совместно

3.2. Проверить ожидаемые пути<br>
Скрипты жёстко рассчитывают на следующие пути:<br>
```bash
assets:
/var/www/r7-office/admin/assets
index.html:
/var/www/r7-office/admin/index.html
nginx config:
/etc/nginx/sites-available/admin
```
Если у вас нестандартная установка — требуется адаптация скрипта.

3.3. Подготовить доступ к mailserver_api<br>
Понадобится:<br>
```txt
IP почтового сервера (можно текущую ВМ, если mailserver_api локально)
порт mailserver_api (по умолчанию 8084)
JWT токен супер-администратора (обычно берётся из /opt/r7-office/mailserver_api/jwt_token)
```
##### 4) Запуск скрипта<br>
4.1. Установка<br>
```bash
unzip plugin-mail-astra174.zip -d plugin-mail-astra174 && cd plugin-mail-astra174
chmod +x r7-mail-manager-install.sh
sudo ./r7-mail-manager-install.sh
```
>[!Warning]
>Скрипт требует root
>Запуск показывает меню, в котором можно Установить/обновить плагин, Переустановить, Удалить, посмотреть статус (если плагин установлен) и выйти из
режима установки

<img width="1228" height="562" alt="image" src="https://github.com/user-attachments/assets/5a8d84c2-3788-4a18-9091-f681b109fe63" />

Далее выбираем онлайн или оффлайн установку (при оффлайн установке пакеты ставятся автоматически из pkg/)

<img width="1226" height="415" alt="image" src="https://github.com/user-attachments/assets/33bf86ee-6227-47c2-ba8b-dbb4a656d7df" />

Следующим шагом скрипт запрашивает IP почтового сервера. Если почта установлена локально, то по умолчанию используется ``127.0.0.1``. Просто нажимаем ``Enter``

<img width="1227" height="339" alt="image" src="https://github.com/user-attachments/assets/8b6b1b03-4ed4-4d79-a927-136553a5cf27" />

Следующий шаг предлагает выбрать порт для сервиса плагина. При выборе порта происходит проверка на его занятость и при недоступности попросит
выбрать другой порт. Просто нажимаем ``Enter``

На следующем шаге требуется установить пароль пользователя admin в htpasswd, от которого будут поступать API запросы.

<img width="1226" height="146" alt="image" src="https://github.com/user-attachments/assets/11d9f88d-fa7f-4837-9796-211fe6267ea1" />

Далее будет установка, проверены все необходимые установленные пакет и зависимости.

<img width="1225" height="790" alt="image" src="https://github.com/user-attachments/assets/b2db1171-f352-4fe5-aa8b-7e81f0eef561" />

После установщик ищет JWT токен (если не будет найден, то попросит его ввести в ручную), устанавливает зависимости, копирует файлы в нужные
директории и вносит изменения.
В конце установки появится справочная информация

<img width="1227" height="686" alt="image" src="https://github.com/user-attachments/assets/f4082721-57dc-4944-8676-dd5d98469285" />

Проверим статус

<img width="1225" height="497" alt="image" src="https://github.com/user-attachments/assets/c1e1457a-8655-49ff-9b80-26a1ed6f2390" />

В файле ``/var/log/r7-mail-manager-install.log``можно просмотреть за всё время все установки которые были выполнены
``cat /var/log/r7-mail-manager-install.log`` или так ``nano /var/log/r7-mail-manager-install.log``

<img width="1222" height="1204" alt="image" src="https://github.com/user-attachments/assets/2c35e7f2-7882-405b-bd28-9da71368007c" />

4.2. Проверка после установки<br>
###### LDAP
```bash
systemctl status r7-mail-manager --no-pager
journalctl -u r7-mail-manager -n 200 --no-pager
nginx -t
systemctl reload nginx
```
Проверка backend через nginx:
###### NGINX
```bash
curl -k https://<HOST>/mail-manager/api/info
```
Проверка UI:<br>
открыть ``админ-панель``<br>
перейти в ``Settings / Настройки``<br>
должна появиться вкладка ``Почта / Mail``<br>
4.3. Восстановление после обновления основного ПО (guard)<br>
Проблема: после обновления (установки патчей) могут пропасть:
```bash
nginx-блоки в /etc/nginx/sites-available/admin
<script> в /var/www/r7-office/admin/index.html
файлы в /var/www/r7-office/admin/assets/
```
>[!Note]
>Решение: guard-скрипт лежит в /opt/r7-mail-manager/ensure-admin-assets.sh и запускается на каждый старт/рестарт сервиса (через ExecStart
Pre):<br>
проверяет целостность nginx-блока # BEGIN R7_MAIL_MANAGER ... # END R7_MAIL_MANAGER<br>
проверяет index.html и наличие inject-*.js<br>
восстанавливает из /opt/r7-mail-manager/backup/<br>





