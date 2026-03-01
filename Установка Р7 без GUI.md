### Установка Корпоративного сервера 2024 без графического интерфейса offline-версии(без интернета) для ОС Astra.
Ссылка на установку и настройку от Р7 

https://support.r7-office.ru/corporate-server2024/install-r7server/single/otkazoustojchivaja-arhitektura-korporativnyj-server-2024-pochtovyj-server-na-astra-linux-1-7-4/

### Установка производится на VM Ware Workstation, версия ОС Астра 1.7.4

Установка состоит из нескольких этапов:

1. Предварительная подготовка.
2. Установка необходимых служб и баз данных для почтового сервера 2024 offline-версии.
3. Установка почтового сервера R7.
4. Настройка почтового сервера.
5. Настройка почтовых протоколов в DNS на доменном сервере ALD PRO.
6. Настройка API/служб Dovecot и Postfix.
7. Настройка почтового клиента Органайзер на клиентском компьютере.
8. Работа в Органайзере.

#### 1. Предварительная подготовка.   

Скачаваем образ `1.7.4-24.04.2023_14.23.iso`

#### Системные требования.
```bash
Поддерживается только 64-битная архитектура 
2 Виртуальные машины
Для хранения данных glusterfs
ТХ Машин, для тестирования, возможно использовать:
- от 4 CPU;
- от 8Гб RAM;
- от 50Гб свободного пространства на диске;
Более конкретные данные рассчитываются по обращению в ТП
Отключение или перевод selinux в режим permissive для корректной работы сервисов
```
>[!WARNING]
>Минимальные системные требования рассчитаны при одновременной работе до 300 пользователей.

Устанавливаем виртуальную машину ОС Астра 1.7.4 из образа
```bash
1.7.4-24.04.2023_14.23.iso`
```
При установке операционной системы выберите пункты <br>
**«Средства работы с интернет»** <br>
**«Консольные утилиты»** <br>
**«Средства удаленного подключения SSH»** <br>
во избежание сообщений об отсутствующих зависимостях <br>

Для упрощения в работе временно даём разрешение на подключению по ssh для суперадмина root

Открываем терминал меняем пароль на root и даём разрешение на подключение в файле `/etc/ssh/sshd_config`
```bash
$ sudo -i
# sed -i 's/^#*PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
```
<img width="1262" height="564" alt="image" src="https://github.com/user-attachments/assets/793d1252-83bc-48c4-baa1-52f988d5add6" />

##### Перезапустить SSH службу для применения изменений:
######  Для системы с systemd (большинство современных дистрибутивов)
```bash
# systemctl restart sshd
```
##### Проверить, что изменения применились:
```bash
# sshd -T | grep permitrootlogin
```
Если есть доступ к общим папкам по сети, тогда подключаем сетевую общую шару. Выполняем команду:
```bash
# mkdir /mnt/myshare
# mount -t cifs //путь до вашей шары  /mnt/myshare –o username=user,password=pass,wr,nounix,iocharset=utf8
```

Открываем репозитории `` nano /etc/apt/sources.list`` и делаем так:

<img width="1278" height="210" alt="image" src="https://github.com/user-attachments/assets/0263381c-bff4-4a27-8aee-9d15cfe0ee08" />

```bash
# sed -i '/^deb.*cdrom/s/^deb/#deb/' /etc/apt/sources.list
# sed -i '/repository-main/s/^#//' /etc/apt/sources.list
# sed -i '/repository-base/s/^#//' /etc/apt/sources.list
# sed -i '/repository-extended/s/^#//' /etc/apt/sources.list
```
Для корректной установки почтового сервера весь архив поместить в директории `/mnt` и распоковать их
```bash
# cd /mnt
```
Скачайте архив Корпоративный сервер 2024 для установки 

- Оффлайн установкаТекущая версия: 2.0.2024.14752
- Модуль Р7 Диск, Модуль Р7-Админ,
- Модуль Р7-Календарь, Модуль Р7-Почта, Модуль Р7-Проекты

![image](https://github.com/user-attachments/assets/5a70f184-9664-4d04-b2b3-9983295d8222)

Ссылка: [https://r7-office.ru/downloadserver#retail]

![image](https://github.com/user-attachments/assets/46376d55-c05c-464a-8b1a-53e62ee45c6e)

![image](https://github.com/user-attachments/assets/495b298b-52d6-437e-b584-4ac6d80fbcec)

![image](https://github.com/user-attachments/assets/3a9287b7-68d2-4bbe-a8a3-9136fa1fddd5)

Скачайте архив `cddisk.zip` который содержит пакеты для корпоративного сервера 2024. 

Ссылка: [https://nct.r7-office.ru/link/549705e3-5e21-49f1-afc4-57c88ecae674]

![image](https://github.com/user-attachments/assets/df7ca167-4541-432a-b97f-8f7a8e98d9ff)

Перетащите его в директорию /mnt  

Зайдите на терминале в директорию со скаченными архивами и распакуте их в корень папки `/mnt`

Распакуйте архив `CDinstall_2.0.2024.14752_Astra_1.7.4_offline.zip`
```bash
# cd /mnt
# unzip CDinstall_2.0.2024.14752_Astra_1.7.4_offline.zip
# unzip cddisk.zip
```
![image](https://github.com/user-attachments/assets/d8f008ef-6e3e-4171-9c89-b628d56b03ad)

![image](https://github.com/user-attachments/assets/4db0bdde-9d56-47c1-a66d-b6a46219d129)

![image](https://github.com/user-attachments/assets/d0403d17-7481-4793-afb7-282de1c4643e)

После как распаковали получились две папки ``CDDiskPack`` и ``cddisk``

Перейдите в каталог ``cddisk`` и установите пакеты командой ``install.sh``
```
# cd cddisk
# chmod +x install.sh
# ./install.sh
```
![image](https://github.com/user-attachments/assets/6f188292-8344-4c91-8724-a3a2ea93e269)

![image](https://github.com/user-attachments/assets/01c8ed7c-29e1-46a8-80a4-2cb419f00b44)

Проверяем что распаковалось всё без ошибок и продолжаем установку.

>[!WARNING]
>Для offline установки требуется установить пакеты в систему и подключить iso образ установочного диска операционный системы в папку distr.

![image](https://github.com/user-attachments/assets/b75c7e92-c332-4eae-a2f6-b865d5333dff)


>[!WARNING]
>Для корректной работы Корпоративного сервера обязательно требуется настройка HTTPS. Перед установкой, скопируйте crt и key файлы в папку ``/mnt/CDDiskPack/CDinstall_Astra_1.7.4/sslcert``. 

Потребуется использовать ssl сертификат типа wildcard с соответствующей А записью (пример, ``*.yourdomain.ru``) на используемом DNS сервере в сети сервера. 

Имя файла должно содержать название домена и расширение. Рекомендуем в .crt указывать всю цепочку сертификатов, домен, промежуточные и корневой. Например, для домена r7.ru имена файлов должны быть ``it.company.lan.crt и it.company.lan.key``.

![image](https://github.com/user-attachments/assets/c2c46672-701f-477e-b184-8817dbf51fd9)

Отключаем полностью доступ в интернет в файле nano ``/etc/network/interfaces`` закомментируем шлюз

![image](https://github.com/user-attachments/assets/bb502064-23a8-46ff-882a-6e3974a611a1)

Проверяем

![image](https://github.com/user-attachments/assets/725be3cb-fe20-4b09-9aac-a2fcc16c3763)

### 2. Установка необходимых служб и баз данных для почтового сервера 2024 offline-версии

Теперь запускаем установку

```bash
# cd /mnt/CDDiskPack/CDinstall_Astra_1.7.4/
# chmod +x offline-installer.sh
# ./offline-installer.sh
```
![image](https://github.com/user-attachments/assets/caa684ee-d0b8-4618-bad7-ab36eda8d02b)

<hr>

<img width="603" height="201" alt="image" src="https://github.com/user-attachments/assets/dc602a12-ffdb-4391-bbf6-6d2051e54af1" />

Чистая установка

Если требуется выполнить чистую установку (удалит имеющуюся инсталляцию Корпоративный сервер 2024 и зависимости):

Выберите Да

<hr>

<img width="601" height="200" alt="2" src="https://github.com/user-attachments/assets/ba377d19-49d5-4821-84d8-87f4b5841fd3" />
 
Установка СУБД на локальную ВМ

Для установки PostgreSQL (при инсталляции всё в одном) на локальный компьютер:

Выберите Да

<hr>

<img width="599" height="200" alt="3" src="https://github.com/user-attachments/assets/4218f9f4-177d-447d-a976-af83b2eafbf6" />

Установка Сервера Документов

Для его установки: Выберите Да

<hr>

<img width="602" height="200" alt="4" src="https://github.com/user-attachments/assets/e3b6a232-e07f-42d7-87b0-a278a64931c3" />

Необходимо ввести пароль-секрет для защищённого доступа между Корпоративный сервер 2024 и Сервером Документов.

<hr>

<img width="597" height="198" alt="5" src="https://github.com/user-attachments/assets/63603451-85eb-48bd-b5b7-492538b4b5db" />


Пароль для Базы Данных DS

<hr>

<img width="599" height="198" alt="6" src="https://github.com/user-attachments/assets/bd2cf086-19e1-4706-9974-7c442186251c" />


Установка api и web диска

Для его установки: Выберите Да

<hr>

<img width="599" height="298" alt="7" src="https://github.com/user-attachments/assets/e09d26da-ab7e-4df5-8493-ab8ccbda422b" />


Тип СУБД Корпоративный сервер 2024

Выберите PostgreSQL

<hr>

<img width="611" height="208" alt="8" src="https://github.com/user-attachments/assets/470ff17f-faef-4d7e-9dff-c82b962b5d6d" />


Создание БД. Выберите Да

<hr>

<img width="604" height="204" alt="9" src="https://github.com/user-attachments/assets/2b344f76-383f-4540-b8ca-d8e8caf9a19e" />


Хост СУБД

При локальной установки. Выберите Ок

>[!Warning]
>Если СУБД установлена отдельно, укажите ip или имя хоста

<hr>

<img width="612" height="206" alt="10" src="https://github.com/user-attachments/assets/ae9f5a21-ce4d-4da7-8303-569d5799ef03" />


По умолчанию 5432 используется. Если настроен другой, указать его. Оставляем как есть.

<hr>

<img width="600" height="199" alt="11" src="https://github.com/user-attachments/assets/a5a91ca5-1971-44a7-920b-b3afb76c0e58" />


Пользователь с правами создания БД и пользователей

По умолчанию ccdisk

<hr>

<img width="604" height="206" alt="12" src="https://github.com/user-attachments/assets/8a9be3b2-3132-4646-a107-fe9947369f39" />


Пароль пользователя с правами создания БД и пользователя ccdisk

<hr>

<img width="621" height="233" alt="13" src="https://github.com/user-attachments/assets/62e7cdd4-02a8-4398-9750-c81bae2035e5" />


Пароль для пользователя БД Корпоративный сервер 2024

Укажите пароль для пользователя cddisk

<hr>

<img width="631" height="234" alt="14" src="https://github.com/user-attachments/assets/d4f96d4f-c4f2-4e7c-9805-eb14e7575be9" />


coremachinkey от CS

Изменить на актуальный, если есть Р7-Офис Корпоративный сервер и нажмите ОК, если нет, нажмите Ок без редактирования

нажмите Ок

<hr>

<img width="602" height="203" alt="image" src="https://github.com/user-attachments/assets/637e435b-a010-4d7e-8259-047ec2fcda0f" />

>[!Warning]
>Если Вы выбрали установки без HTTPS, то, после инсталляции, почтовый сервер работать не будет. Он не запустится.

<hr>

<img width="605" height="205" alt="15" src="https://github.com/user-attachments/assets/fe61163e-5696-4b4d-8a78-e25006facfd4" />


Укажите домен. Необходимо указать домен, в котором у Вас созданы записи

Например, it.company.lan

<hr>

<img width="606" height="204" alt="16" src="https://github.com/user-attachments/assets/12787eb0-c4e8-4fe6-9e66-2d8298461330" />


Префикс Р7-Диск

Указать имя, которое будет открываться в браузере для веб р7-Диска

Например, если Вы хотите, чтобы открылся Р7-Диск по адресу cdisk.it.company.lan, то указать нужно именно cdisk, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

<img width="605" height="206" alt="17" src="https://github.com/user-attachments/assets/4607d0b4-bfb9-4c73-a009-065350a7b804" />


Префикс Р7-Админ

Указать имя, которое будет открываться в браузере для веб админской панели.

Например, если Вы хотите, чтобы открылся Р7-Админ по адресу admin.it.company.lan, то указать нужно именно admin, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

<img width="603" height="201" alt="18" src="https://github.com/user-attachments/assets/3c4c8c6d-1c30-4d8c-b11b-a4471bc1a9e4" />


Префикс Р7-Контакты

Указать имя, которое будет открываться в браузере для веб клиента контактов.

Например, если Вы хотите, чтобы открылся Р7-Контакты по адресу contacts.it.company.lan, то указать нужно именно contacts, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

<img width="602" height="201" alt="19" src="https://github.com/user-attachments/assets/36bd439c-c4af-4cec-8440-c0efa6d2737c" />


Префикс Р7-Почта

Указать имя, которое будет открываться в браузере для веб клиента почты.

Например, если Вы хотите, чтобы открылся Р7-Почта по адресу objects.it.company.lan, то указать нужно именно objects, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

<img width="606" height="209" alt="20" src="https://github.com/user-attachments/assets/2fc3b474-d5fa-40c2-830c-cab2e0d34251" />


Префикс Р7-Почта

Указать имя, которое будет открываться в браузере для веб клиента почты.

Например, если Вы хотите, чтобы открылся Р7-Почта по адресу cmail.it.company.lan, то указать нужно именно cmail, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

<img width="603" height="203" alt="21" src="https://github.com/user-attachments/assets/30150f6b-0059-4516-92ba-dfa80f2390d1" />


Префикс Р7-Календарь

Укажите имя, которое будет открываться в браузере для веб календаря.

Например, если Вы хотите, чтобы открылся Р7-Календарь по адресу calendar.it.company.lan, то указать нужно именно calendar, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

<img width="599" height="198" alt="22" src="https://github.com/user-attachments/assets/1eee1676-f0cb-408f-9776-bf001bb328f4" />


Префикс Р7-Сервер Документов

Указать имя, которое будет открываться в браузере для веба сервера документом.

Например, если Вы хотите, чтобы открылся Р7-Сервер Документов по адресу ds.it.company.lan, то указать нужно именно ds, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

### 3. Установка почтового сервера R7.

<img width="603" height="199" alt="23" src="https://github.com/user-attachments/assets/bcc57017-9477-420b-b2dc-2415bc1061e8" />


Установка почтового сервера - ДА

---

<img width="600" height="292" alt="24" src="https://github.com/user-attachments/assets/a4e5b28c-5746-4aa3-a1f6-d5f5b69543a0" />


Выберите базу данных для установки.

---

<img width="602" height="200" alt="25" src="https://github.com/user-attachments/assets/1a862468-36cc-4035-88d7-449750df5b88" />


В данном окне надо внести имя вашего MX почтового сервера, который прописан в записи А на контроллере домена. 

В моём случае запись А в DNS такая ``A  r7mx1.it.company.lan  TTL``

Если MX запись выглядит, как r7mx1.it.company.lan, то ввести необходимо просто hostname почтового сервера **r7mx1**

<hr>

<img width="606" height="195" alt="26" src="https://github.com/user-attachments/assets/0fd8c74c-f00f-4fda-bcfc-2b7eaa1fb43c" />


Укажите ip адрес

Укажите внешний ip адрес сервера, для корректной работы почтового сервера.

Если указать приватный внутренний ip, то почта будет работать только внутри сети организации.

<hr>

<img width="605" height="199" alt="27" src="https://github.com/user-attachments/assets/2b3d2b6f-3e0d-4b0c-a09a-439da949ed46" />


Пароль root для Postgresql

Укажите пароль, который будет задан пользователю root Postgresql

<hr>

<img width="609" height="307" alt="28" src="https://github.com/user-attachments/assets/d981e10d-fb78-4d1c-ac8f-58f5721cc20f" />


Установка SpamAssassin

Если требуется установка

Выберите 1

<hr>

<img width="604" height="197" alt="29" src="https://github.com/user-attachments/assets/a58089cd-9974-42e8-a134-b4d332675c63" />


Установка OpenDKIM

Если требуется установка. Выберите Да



<hr>

**DKIM** — это технология аутентификации электронной почты, которая позволяет проверить, что письмо действительно отправлено от имени домена и не было изменено в процессе доставки. DKIM использует цифровую подпись, чтобы подтвердить подлинность отправителя и целостность содержимого письма.<br>
DKIM-запись хранится в DNS домена в формате TXT-записи и выглядит примерно так:
```bash
# selector._domainkey.it.company.lan.  TXT  "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC..."
```
После инсталляции в консоли будет предложено сделать TXT запись.

![image](https://github.com/user-attachments/assets/29b8a56f-4da1-46e7-acaf-4a4b4c1814a2)

**Перезагружаем севрвер**

Проверяем работу службы ``rabbitmq``.

![image](https://github.com/user-attachments/assets/3672e338-7388-4579-9480-13eba974e74d)

Обратно раскомментируем шлюз и репозитории.

![image](https://github.com/user-attachments/assets/228cff78-e77b-4ff6-8ead-7fe020f835c9)

![image](https://github.com/user-attachments/assets/815645a7-c5d9-4f2e-b576-48b409844458)

### 4. Настройка почтового сервера Р7.

Открываем браузер Мозилла, вводим адрес почтового сервер, чтобы попасть в панель администрации.
```bash
# https://admin.it.company.lan
```

![image](https://github.com/user-attachments/assets/4ebab8ed-fbfc-4723-a2a0-dbafa6bfbfcc)

![image](https://github.com/user-attachments/assets/00519bae-6323-4e46-bca9-7deb495ae38a)

![image](https://github.com/user-attachments/assets/fb373ca0-d024-4850-b169-b67b52f10c36)

![image](https://github.com/user-attachments/assets/e0f61e64-b714-4a3e-a7a4-8ed41c8d2523)

![image](https://github.com/user-attachments/assets/e1c6ca11-968a-4dc7-91e4-0877cbd16fb3)


В настройках КС24 указать корректные порты ПС.

- **POP сервер** - 995  Тип соединения: None 
- **IMAP сервер** - 993 Тип соединения: SslOnConnect 
- **SMTP сервер** - 465 Тип соединения: SslOnConnect

Создать почтового пользователя и проверить работу почты.

Переходим во вкладку  ![image](https://github.com/user-attachments/assets/a143ef65-fa75-4438-9fd0-d4f45d8cd2c1) ``Почта``

![image](https://github.com/user-attachments/assets/0490e089-4f78-4ea4-b219-9c27a88d009e)

![image](https://github.com/user-attachments/assets/ef89b633-2f21-40aa-8123-2898c7222516)

![image](https://github.com/user-attachments/assets/0120fac9-ad57-4cb0-b1f5-d65358016b5c)

![image](https://github.com/user-attachments/assets/c78b0506-fea7-4f9b-a230-9bf73546c794)

>[!Warning]
>После установки почтового сервера прописываем записи в DNS для работы почтового сервера.
>Необходимо добавить SRV записи IMAP, IMAPS, POP, SMTP и AUTODISCOVER для автоматического обнаружения протоколов. Создать файл ``autodiscover.xml``.

Остальные настройки находятся в файлах.

- Настройка в файле Autodiscover.md
- Настройка SRV записей.md
- Настройка почтовых протоколов.md
- Настройка API.md
- Установка Органайзера.md
- Установка модуля графики.md




















