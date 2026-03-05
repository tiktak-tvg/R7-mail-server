### Плагин «SSL – Мониторинг»

Плагин работает в режиме реального времени и представляет собой информацию о SSL сертификате, который используется на Корпоративном Сервере.

На странице плагина находится:

Описание текущего сертификата в формате «Ключ – Значение»

<img width="1296" height="367" alt="image" src="https://github.com/user-attachments/assets/fb929b91-8b54-453a-ac44-9753ee7320e5" />

Возможность добавить сертификат (.crt) и ключ (.key) с диска устройства, которое используется, а так же пароль ключа если есть:

<img width="1313" height="196" alt="image" src="https://github.com/user-attachments/assets/58200140-f31d-4d22-b0e1-eb602536755e" />

Лог операции:

<img width="1313" height="399" alt="image" src="https://github.com/user-attachments/assets/8b41e4a8-073b-4ae1-aac0-1ee5d7ef4386" />

если ключ не рабочий в логе операций появится ошибка

<img width="1279" height="389" alt="image" src="https://github.com/user-attachments/assets/53950838-1f61-404e-909d-bc3d915bae5c" />

>[!Warning]
>Замена сертификата и ключа выполняется безопасно: старые файлы будут скопированы с префиксом даты рядом с текущими. <br>После успешной проверки nginx проверяется с помощью команды nginx -t и перезагружается.

Для того, что бы воспользоваться функционалом плагина, нужно авторизоваться под учетной записью администратора Корпоративного Сервера с доступом к модулю «Управление» и всеми необходимыми правами, далее перейти в модуль «Управление», открыть вкладку «Общие настройки» и в левом боковом меню выбрать «SSL»:

<img width="1619" height="1151" alt="image" src="https://github.com/user-attachments/assets/88f02c7a-2ecb-4c5d-adbb-499eb8807a86" />

Так же в модуле «Управление», на вкладке «Общее» отображается виджет, который отображает информацию:

- Wildcard – тип сертификата SSL<br>
- Дата, до которой действует сертификат<br>
- Кол-во дней до окончания действия сертификата<br>
- Статус-бар по длительности сертификата<br>

<img width="1597" height="371" alt="image" src="https://github.com/user-attachments/assets/4f1b47b2-3afb-4024-9114-3ce6e783690e" />

### Установка плагина SSL

Скачать плагин можно по [ссылке](https://download.r7-office.ru/plugin/plugin-ssl.zip) <br> 
MD5: E3D535A683682823A3E3F86A0C293286

#### 1. Список необходимых пакетов
Для ВМ без внешних репозиториев и без доступа в интернет.

1.1. Что должно быть установлено (пакеты / команды)<br>
На сервере Корпоративный сервер 2024.

Обязательно:
```txt
nodejs
Backend-плагин написан на Node.js;
Сервис запускает node *.js;
Версия ≥ 10 (рекомендуется 16+).
```
>[!Warning]
> - Установщик пытается установить nodejs сам, но в офлайне рекомендуется установить заранее системным пакетом
> - Во избежании ошибок, можете сразу установить ``apt install nodejs``


Дополнительно (обычно уже есть в системе, но важно):
```
openssl
ss (из пакета iproute2)
```
Используется для проверки занятости портов

Скрипт активно:
```
Патчит index.html;
Модифицирует nginx-конфиг;
Создает systemd-unit.
```
#### 2. Назначение скрипта
```bash
r7-ssl-install.sh
```
Скрипт выполняет установку / обновление / удаление SSL Monitor:
```txt
Разворачивает backend в: /opt/r7-ssl-status
Создаёт systemd-сервис: r7-ssl-status r7-ssl-change
```
Копирует JS-инжекты:
```
inject-ssl-status.js
inject-ssl-tab.js
```
Патчит index.html админки;

Добавляет nginx location:
```
/ssl-status
/ssl-change
```
Проверяет и запрашивает порты.

Поддерживает удаление с выбором:

Оставить или удалить inject-manager.js

#### 3. Подготовка перед запуском
3.1. Подготовить файлы рядом со скриптом

##### SSL Monitor.
```bash
r7-ssl-install.sh

data/

├── inject-manager.js
├── inject-ssl-status.js
├── inject-ssl-tab.js
└── ssl-status/
──├── index.html
──├── ssl-status.js
──└── ssl-change.js
```
>[!Warning]
>- inject-manager.js копируется только если его ещё нет используется всеми плагинами совместно

3.2. Проверить ожидаемые пути

Скрипты жёстко рассчитывают на следующие пути:
```
assets: /var/www/r7-office/admin/assets
index.html: /var/www/r7-office/admin/index.html
nginx config: /etc/nginx/sites-available/admin
```
Если у вас нестандартная установка — требуется адаптация скрипта.

#### 4. Запуск скрипта
4.1. Установка
>[!Warning]
>- inject-manager.js копируется только если его ещё нет используется всеми плагинами совместно

3.2. Проверить ожидаемые пути

Скрипты жёстко рассчитывают на следующие пути:
```
assets: /var/www/r7-office/admin/assets
index.html: /var/www/r7-office/admin/index.html
nginx config: /etc/nginx/sites-available/admin
```
Если у вас нестандартная установка — требуется адаптация скрипта.

#### 4. Запуск скрипта
4.1. Установка
```bash
unzip Plugin-ssl.zip -d r7-ssl-monitor && cd r7-ssl-monitor
chmod +x r7-ssl-install.sh
sudo ./r7-ssl-install.sh
```
Запуск сразу указывает на ключ —info для получения описания скрипта и предлагает сменить пароль, если предлагаемый порт занят или выбрать по умолчанию: <br>
если ставите впервые, можно без ввода сразу нажать Enter, далее происходит установка и вывод конечного результата:

<img width="1216" height="635" alt="image" src="https://github.com/user-attachments/assets/301cea85-622e-4e51-978b-955e445ec245" />

Во время установки:<br>
- Скрипт требует root;<br>
- Запрашивает порт(ы);<br>
- Проверяет, свободны ли они;<br>
- При конфликте предлагает выбрать другой;<br>
- Выполняет установку;<br>
- Выводит финальный статус.<br>
4.2. Повторный запуск<br>
При повторном запуске:<br>
- Выполняется обновление;<br>
- Существующие файлы корректно перезаписываются;<br>
- inject-manager.js не дублируется;<br>
- Порядок инжектов сохраняется;<br>
- Порты придётся сменить.<br>
<img width="1222" height="95" alt="image" src="https://github.com/user-attachments/assets/d20f0817-d7da-4f91-ab42-aa71354e037b" />

4.3. Проверка после установки
```bash
systemctl status r7-ssl-status r7-ssl-change --no-pager
journalctl -u r7-ssl-status -n 200 --no-pager
nginx -t
systemctl reload nginx
```
<img width="1222" height="911" alt="image" src="https://github.com/user-attachments/assets/950e3b52-4932-469f-9fa1-0f7861d520a1" />

4.4. Удаление
##### SSL:
```bash
sudo ./r7-ssl-install.sh --uninstall
```
Во время удаления:<br>
- Сервис останавливается и удаляется;<br>
- Backend-каталоги очищаются;<br>
- Nginx-патчи удаляются.<br>
- Будет задан вопрос:<br>
>[!Warning]
>- Удалять ли inject-manager.js? выбираем N . Это важно, если установлены другие плагины.

#### 5. Порядок работы скрипта (последовательность)
**Установка:** <br>
- Проверка прав root;<br>
- Проверка / запрос портов;<br>
- Проверка занятости портов (ss);<br>
- Проверка Node.js (установка при необходимости)<br>
- Проверка структуры файлов;<br>
- Копирование backend в /opt/...<br>
- Создание systemd-unit’ов (inline);<br>
- systemctl daemon-reload;<br>
- Включение и запуск сервисов;<br>
- Копирование JS в assets;<br>
- Патч index.html(manager → plugin-инжекты)<br>
- Патч nginx (location + websocket);<br>
- nginx -t и reload;<br>
- Финальный отчёт и статус сервисов.<br>
#### 6. Как работают плагины (архитектура)
6.1. Общая схема<br>
- Пользователь работает только через админку R7 Office.<br>
**Админка:** <br>
- Загружает JS-инжекты;<br>
- Отправляет HTTP / WS-запросы.<br>
**nginx:** <br>
- Проксирует запросы на localhost.<br>
**backend:** <br>
- Обрабатывает API;<br>
- Отдаёт данные интерфейсу.<br>
6.2. inject-manager.js<br>
- Общий менеджер вкладок / карточек;<br>
- Подключается один раз.<br>
**Все плагины:** <br>
- Вставляются ниже него;<br>
- Не дублируют его установку.<br>
6.3. SSL Monitor<br>
**backend:**
```bash
ssl-status.js
ssl-change.js
```
**UI:** <br>
Вкладка SSL<br>
- nginx проксирует: /ssl-status /ssl-change
#### 7. Что именно меняется в системе (для аудита)
Установка затрагивает:
```bash
/opt/r7-ssl-status/*
/etc/systemd/system/*.service
/etc/nginx/sites-available/admin
/var/www/r7-office/admin/assets/*.js
/var/www/r7-office/admin/index.html
```
Для критичных файлов создаются .bak-копии.





