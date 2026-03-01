## Установка Корпоративного сервера 2024 с графическим интерфейсом offline-версии(с интернетом) для ОС Astra.

Установка произваодится на VM Ware Workstation, версия ОС Астра 1.7.4

Установка состоит из нескольких этапов:

1. Предварительная подготовка.
2. Установка необходимых служб и баз данных для почтового сервера 2024 offline-версии.
3. Установка почтового сервера R7.
4. Настройка почтового сервера.
5. Настройка почтовых протоколов в DNS на доменном сервере ALD PRO.
6. Настройка API/служб Dovecot и Postfix.
7. Настройка почтового клиента Органайзер на клиентском компьютере.
8. Работа в Органайзере.

### 1. Предварительная подготовка.   

Устанавливаем ОС Астра 1.7.4 образ 1.7.4-24.04.2023_14.23.iso

После установки выполняем команды:

Открываем репозитории `` nano /etc/apt/sources.list``

<img width="1278" height="210" alt="image" src="https://github.com/user-attachments/assets/26437219-ce37-4ca5-85d4-f40563ca7bf3" />

```bash
apt install open-vm-tools
```
```bash
apt update
apt install astra-update
apt install dnsutils
```
Всё, больше никаких манипуляций с обновлениями.

Проверим версию ``cat /etc/astra/build_version``

Должна быть 1.7.4.7

![image](https://github.com/user-attachments/assets/7050daa1-c528-4897-b120-892af3d87965)

Если версия такая, идём дальше.

Добавим и запустить службу синхронизации времени chrony в автозапуск.
>[!Warning]
>Серверная служба chronyd (пакет chrony) Может обеспечивать работу ОС в режиме как сервера точного времени, так и клиента. Является штатной службой времени для использования с контроллерами домена FreeIPA.

Служба chrony по умолчанию не установлена, установить можно так
```bash
sudo apt install chrony
sudo systemctl start chrony
sudo systemctl enable chrony
```
>[!Warning]
>Так как порт закрыт, если слушает порт udp 123: ``sudo ss -ulpn | grep chronyd`` - тогда не надо.
>Нужно, прописать наш доменный сервер в качестве источника синхронизации времени.  
>Для этого укажем его адрес в ``/etc/systemd/timesyncd.conf`` на остальных серверах. 
```bash
[Time]
NTP=192.168.25.100
А затем перезапустим службу синхронизации времени ``systemctl restart chrony``
```
Делаем статический адрес. Отключаем NetworkManager.

Переписываем свой IP дарес, шлюз, маску, не ошибиться при настройке статического IP-адреса.
```bash
nmcli dev show
```
![image](https://github.com/user-attachments/assets/e4dfa035-87bb-48e1-8a0e-11a11db3717f)

Откючаем и забываем, на серверах не должно быть динамического адреса.
```bash
sudo systemctl status NetworkManager //проверяем статус службы NetworkManager
sudo systemctl stop NetworkManager //останавливает службу
sudo systemctl disable NetworkManager //удаляет её из автозагрузки
sudo systemctl mask NetworkManager //останавливает активность службы
astra-noautonet-control disable //контрольный выстрел
```
![image](https://github.com/user-attachments/assets/27f2a9f5-b55a-44fc-ba41-1345cd231627)

Настраиваем статический адрес службы ``networking.service`` вводим команду: ``nano /etc/network/interfaces``
```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

auto eth0
allow-hotplug eth0
iface eth0 inet static
address 192.168.25.91
netmask 255.255.255.0
gateway 192.168.25.10
```
![image](https://github.com/user-attachments/assets/00d2ccda-055e-444a-8853-c4aebd40b6dc)

Чтобы применить новые настройки, перезагружаем машину:
```bash
reboot
```
Если всё правильно сделали, то ответ на введённую команду ``sudo systemctl status NetworkManager`` после перезагрузки ответ будет таким, пока не перезагружаем:

![image](https://github.com/user-attachments/assets/25a62ca3-5814-47af-96a0-37f04065a4f2)

После загрузки, проверяем
```bash
ping 77.88.8.8 -c 4
```
![image](https://github.com/user-attachments/assets/f3779d53-5032-45f7-bdab-ffa28a56d2ab)

Если пинга нет, смотрите прописаны ли DNS записи в файле ``nano /etc/resolv.conf``. 

Теперь в файл hosts добавим строки с именем сервера ``nano /etc/hosts``.
```bash
127.0.0.1        localhost.localdomain   localhost
# 127.0.1.1      r7mx1.it.company.lan    r7mx1   --обязательно закомментировать
192.168.25.91    r7mx1.it.company.lan    r7mx1
192.168.25.201   dc01.it.company.lan     dc01 

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
![image](https://github.com/user-attachments/assets/27ed8d1f-1a38-48f6-af36-1c3fe0e211a2)

Настраиваем ``FQDN`` имя первого контроллера домена:
```bash
hostnamectl set-hostname r7mx1.it.company.lan
```
Перезапустим сетевой интерфейс для применения настроек
```bash 
systemctl restart networking.service
```
Проверяем
```bash
hostname -s
hostname -f // если не работает проверяем запись в файле etc/hosts
```
Проверяем resolv.conf
```bash
cat /etc/resolv.conf
search it.company.lan
nameserver 77.88.8.8
nameserver 192.168.25.201 //наш контроллер домена, где настроен DNS
```
![image](https://github.com/user-attachments/assets/4ffaca52-b682-4ac0-bab1-75a39456d7c5)

#### Установка Корпоративного сервера 2024 offline-версии. 

Скачиваем с их сайта архив **Оффлайн установка** CDinstall_2.0.2024.14752_Astra_1.7.4_offline.zip<br>

> https://r7-office.ru/downloadserver#retail

или с нашей шары и закидываем в корень папки ``/mnt`` переходим в неё и распаковываем
```bash
cd /mnt/
unzip CDinstall_2.0.2024.14752_Astra_1.7.4_offline.zip
```
![image](https://github.com/user-attachments/assets/29d3d29e-d27f-4732-8851-918518dc749f)

После распаковки заходим в папку
```bash
cd /mnt/CDDiskPack/CDinstall_Astra_1.7.4/sslcert
проваливаемся то этому пути
/mnt/CDDiskPack/CDinstall_Astra_1.7.4/sslcert#
```
##### Генерируем самоподписанные сертификаты.
Генерация самоподписанных SSL-сертификатов включает в себя четыре простых шага:

- Шаг 1: Создайте конфигурационный файл (например с названием, openssl.cnf) с поддержкой wildcard:
```bash 
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = RU
ST = Moscow
L = Moscow
O = Rosreestr
CN = *.it.company.lan 

[v3_req]
keyUsage = keyEncipherment, dataEncipherment, digitalSignature
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = *.it.company.lan  
DNS.2 = it.company.lan  
```
- Шаг 2:  Команда создаст 2048-битный закрытый ключ **it.company.lan.key**
  
  в этой папке /mnt/CDDiskPack/CDinstall_Astra_1.7.4/sslcert#
```bash
openssl genrsa -out it.company.lan.key 2048
```

- Шаг 3: Следующая команда создаст сертификат CSR-Certificate Signing Request **it.company.lan.csr** для существующего ключа (it.company.lan.key):
```bash
openssl req -new -key it.company.lan.key -out it.company.lan.csr -config openssl.cnf
```

- Шаг 4: Следующая команда сгенерирует самоподписанный сертификатс **it.company.lan.crt** на основе существующего сертификата (it.company.lan.csr) и закрытого ключа (it.company.lan.key):
```bash
openssl x509 -req -days 365 -in it.company.lan.csr -signkey it.company.lan.key -out it.company.lan.crt -extensions v3_req -extfile openssl.cnf
```

##### Добавляем корневой сертификат в доверенные сертификаты.

Скопируйте файл вашего сертификата (it.company.lan.crt) в хранилище сертификатов в каталог usr/local/share/ca-certificates/:
```bash
мы сейчас долны находиться в той папке где мы создали сертификаты  /mnt/CDDiskPack/CDinstall_Astra_1.7.4/sslcert#
sudo cp it.company.lan.crt /usr/local/share/ca-certificates/
```
Обновите хранилище сертификатов командой:
```bash
update-ca-certificates -v
```
![image](https://github.com/user-attachments/assets/733724e8-53fa-4f38-be70-879a6c4a7ce7)

Если сертификаты успешно добавлены, появится сообщение о том, что сертфикат скопирован в /etc/ssl/certs/:
```bash
Updating certificates in /etc/ssl/certs…
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d
```
##### Проверяем работу сертификатов.

**CSR**

Эта команда проверит CSR и отобразит данные, указанные в запросе:
```bash
openssl req -text -noout -verify -in it.company.lan.csr
```


**Ключ**

Следующая команда проверит ключ и его действительность:
```bash
openssl rsa -in it.company.lan.key -check
```


**SSL сертификат**

Когда вам нужно проверить сертификат, дату его истечения и кто его подписал, используйте следующую команду OpenSSL:
```bash
openssl x509 -in it.company.lan.crt -text 
```


Обновите хранилище:
```bash
update-ca-certificates --fresh
```
Теперь Вы можете убедиться, что ваша ОС доверяет сертификату с помощью команду:
```bash
#openssl verify it.company.lan.crt
```
![image](https://github.com/user-attachments/assets/9d33ea5b-6500-42c0-9f81-555bf97aadd1)


### 2. Установка необходимых служб и баз данных для почтового сервера 2024 offline-версии.
```bash
из папки в которой создали сертификаты /mnt/CDDiskPack/CDinstall_Astra_1.7.4/sslcert/#
переходим 
cd ..
проваливаемся
/mnt/CDDiskPack/CDinstall_Astra_1.7.4/#
смотрим что в папке скрипт присутствует  ls -l
даём права на выполнение его
chmod +x offline_installer.sh
запускаем
./offline_installer.sh
```
![image](https://github.com/user-attachments/assets/caa684ee-d0b8-4618-bad7-ab36eda8d02b)

<hr>

![image](https://github.com/user-attachments/assets/986d1e7b-d33f-4234-8fd7-aed5e9451eda)

Чистая установка

Если требуется выполнить чистую установку (удалит имеющуюся инсталляцию Корпоративный сервер 2024 и зависимости):

Выберите Да

<hr>

![image](https://github.com/user-attachments/assets/9637b5a5-3114-486a-9dba-18c67441ba18)
 
Установка СУБД на локальную ВМ

Для установки PostgreSQL (при инсталляции всё в одном) на локальный компьютер:

Выберите Да

<hr>

![image](https://github.com/user-attachments/assets/878632c3-5ad9-445a-a670-3599e0723062)

Установка Сервера Документов

Для его установки: Выберите Да

<hr>

![image](https://github.com/user-attachments/assets/42f7d7f0-3b6a-494b-a527-a15b7246c2b6)

Необходимо ввести пароль-секрет для защищённого доступа между Корпоративный сервер 2024 и Сервером Документов.

<hr>

![image](https://github.com/user-attachments/assets/2237c1d7-2c86-4491-ae5f-bea349f658cd)

Пароль для Базы Данных DS

<hr>

![image](https://github.com/user-attachments/assets/7cce156d-c595-45fe-b81f-47d202d8555b)

Установка api и web диска

Для его установки: Выберите Да

<hr>

![image](https://github.com/user-attachments/assets/0135cee8-32f3-446d-91df-b1ca94db8e9a)

Тип СУБД Корпоративный сервер 2024

Выберите PostgreSQL

<hr>

![image](https://github.com/user-attachments/assets/3f5778a3-0834-4aa3-a11d-a182a9bb3aa8)

Создание БД. Выберите Да

<hr>

![image](https://github.com/user-attachments/assets/83f23eaf-2540-4dca-a145-1d2c7066b182)

Хост СУБД

При локальной установки. Выберите Ок

>[!Warning]
>Если СУБД установлена отдельно, укажите ip или имя хоста

<hr>

![image](https://github.com/user-attachments/assets/84e27862-4db4-4d62-a272-eaa53aa900d3)

По умолчанию 5432 используется. Если настроен другой, указать его. Оставляем как есть.

<hr>

![image](https://github.com/user-attachments/assets/b8b03d01-4e58-46ef-ac1d-cb19e0719d60)

Пользователь с правами создания БД и пользователей

По умолчанию ccdisk

<hr>

![image](https://github.com/user-attachments/assets/10eff70d-b8db-421e-ad40-69772b77fa5d)

Пароль пользователя с правами создания БД и пользователя ccdisk

<hr>

![image](https://github.com/user-attachments/assets/5afb9ace-7c43-48b1-83ce-22cd7e17e2ed)

Пароль для пользователя БД Корпоративный сервер 2024

Укажите пароль для пользователя cddisk

<hr>

![image](https://github.com/user-attachments/assets/2f2e3341-85de-458d-8aac-40ca5e562733)

coremachinkey от CS

Изменить на актуальный, если есть Р7-Офис Корпоративный сервер и нажмите ОК, если нет, нажмите Ок без редактирования

нажмите Ок

<hr>

![image](https://github.com/user-attachments/assets/79e5b956-5dbf-4db6-8240-77bacc7ed8df)

>[!Warning]
>Если Вы выбрали установки без HTTPS, то, после инсталляции, почтовый сервер работать не будет. Он не запустится.

<hr>

![image](https://github.com/user-attachments/assets/ae23bfe7-4777-4147-ae3a-4776c64e94a2)

Укажите домен. Необходимо указать домен, в котором у Вас созданы записи

Например, it.company.lan

<hr>

![image](https://github.com/user-attachments/assets/6aba3925-df9e-4c4b-ba06-c5dd15954b0d)

Префикс Р7-Диск

Указать имя, которое будет открываться в браузере для веб р7-Диска

Например, если Вы хотите, чтобы открылся Р7-Диск по адресу cdisk.it.company.lan, то указать нужно именно cdisk, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

![image](https://github.com/user-attachments/assets/64ec3ab8-8845-4765-bdf5-59e5af56220d)

Префикс Р7-Админ

Указать имя, которое будет открываться в браузере для веб админской панели.

Например, если Вы хотите, чтобы открылся Р7-Админ по адресу admin.it.company.lan, то указать нужно именно admin, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

![image](https://github.com/user-attachments/assets/b7bfadab-6d3f-4f86-aa85-b9e1306ff9cf)

Префикс Р7-Контакты

Указать имя, которое будет открываться в браузере для веб клиента контактов.

Например, если Вы хотите, чтобы открылся Р7-Контакты по адресу contacts.it.company.lan, то указать нужно именно contacts, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

![image](https://github.com/user-attachments/assets/8135dcfa-eaae-46f7-bb11-27517b0ea0a3)

Префикс Р7-Почта

Указать имя, которое будет открываться в браузере для веб клиента почты.

Например, если Вы хотите, чтобы открылся Р7-Почта по адресу objects.it.company.lan, то указать нужно именно objects, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

![image](https://github.com/user-attachments/assets/02f3207b-6504-493e-a376-f533b36afa0f)

Префикс Р7-Почта

Указать имя, которое будет открываться в браузере для веб клиента почты.

Например, если Вы хотите, чтобы открылся Р7-Почта по адресу cmail.it.company.lan, то указать нужно именно cmail, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

![image](https://github.com/user-attachments/assets/8a648089-4a26-41d2-ba66-47a275ab64cc)

Префикс Р7-Календарь

Укажите имя, которое будет открываться в браузере для веб календаря.

Например, если Вы хотите, чтобы открылся Р7-Календарь по адресу calendar.it.company.lan, то указать нужно именно calendar, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

![image](https://github.com/user-attachments/assets/a3bc029d-20b3-4918-860b-77c4556ccbef)

Префикс Р7-Сервер Документов

Указать имя, которое будет открываться в браузере для веба сервера документом.

Например, если Вы хотите, чтобы открылся Р7-Сервер Документов по адресу ds.it.company.lan, то указать нужно именно ds, без указания домена.

Также, необходимо сделать соответствующую А запись в DNS.

<hr>

### 3. Установка почтового сервера R7.

![image](https://github.com/user-attachments/assets/a25c4ad1-94c2-4995-a5cd-4fecf3de316d)

Установка почтового сервера - ДА

---

![image](https://github.com/user-attachments/assets/609df310-f84f-45ce-ae68-dd27454c3a5d)

Выберите базу данных для установки.

---

![image](https://github.com/user-attachments/assets/9a690a18-2476-4aa7-b307-453becec49da)

В данном окне надо внести имя вашего MX почтового сервера, который прописан в записи А на контроллере домена. 

В моём случае запись А в DNS такая ``A  r7mx1.it.company.lan  TTL``

Если MX запись выглядит, как r7mx1.it.company.lan, то ввести необходимо просто hostname почтового сервера **r7mx1**

<hr>

![image](https://github.com/user-attachments/assets/75998f41-5961-4fe5-ae77-b3eaa6f90573)

Укажите ip адрес

Укажите внешний ip адрес сервера, для корректной работы почтового сервера.

Если указать приватный внутренний ip, то почта будет работать только внутри сети организации.

<hr>

![43](https://github.com/user-attachments/assets/396395fb-010e-4b6d-8c33-a9d1c7243f25)

Пароль root для Postgresql

Укажите пароль, который будет задан пользователю root Postgresql

<hr>

![44](https://github.com/user-attachments/assets/e0cb49e9-6577-422a-b485-954d502aea6e)

Установка SpamAssassin

Если требуется установка

Выберите 1

<hr>

![45](https://github.com/user-attachments/assets/6c0ebd51-4130-436e-92f9-d9af2ddc4be9)

Установка OpenDKIM

Если требуется установка. Выберите Да

<hr>

**DKIM** — это технология аутентификации электронной почты, которая позволяет проверить, что письмо действительно отправлено от имени домена и не было изменено в процессе доставки. DKIM использует цифровую подпись, чтобы подтвердить подлинность отправителя и целостность содержимого письма.<br>
DKIM-запись хранится в DNS домена в формате TXT-записи и выглядит примерно так:
```bash
selector._domainkey.it.company.lan.  TXT  "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC..."
```
После инсталляции в консоли будет предложено сделать TXT запись.

![image](https://github.com/user-attachments/assets/ef1879c9-ed84-4291-a4a5-8ceb5bdb243e)


### 4. Настройка почтового сервера Р7.

Открываем браузер Мозилла, вводим адрес почтового сервер, чтобы попасть в панель администрации.
```bash
https:/admin.it.company.lan
```
![image](https://github.com/user-attachments/assets/d0b2d655-32fa-4980-b66e-7e1a45a3ef53)

![image](https://github.com/user-attachments/assets/033f68e7-afb0-4b50-a156-fa537faa8e2a)

Теперь заходимим в настройки почтововго сервера.

![image](https://github.com/user-attachments/assets/c3b48161-f9de-4e8b-9b76-e241ced3ed70)

![image](https://github.com/user-attachments/assets/96f36de7-533e-4f55-b646-efb705fbee3d)

![image](https://github.com/user-attachments/assets/75b522e4-b430-4aa3-b333-0b4c35eeec54)

В настройках КС24 указать корректные порты ПС.

- **POP сервер** - 995  Тип соединения: None 
- **IMAP сервер** - 993 Тип соединения: SslOnConnect 
- **SMTP сервер** - 465 Тип соединения: SslOnConnect

Создать почтового пользователя и проверить работу почты.

Переходим во вкладку  ![image](https://github.com/user-attachments/assets/a143ef65-fa75-4438-9fd0-d4f45d8cd2c1) `Почта`

![image](https://github.com/user-attachments/assets/1ce65e48-068a-471d-a724-6c3266f85821)

![image](https://github.com/user-attachments/assets/5203e6d2-24e7-4a86-aea3-332a9265bd3c)

![image](https://github.com/user-attachments/assets/ddefa756-6816-451e-aaab-c93a34ddc059)

![image](https://github.com/user-attachments/assets/1e1f79e4-3b5b-4429-9efe-833154cdc8b1)


>[!Warning]
>После установки почтового сервера рекомендую прописать записи в DNS для работы почтового сервера.
>Необходимо добавить SRV записи IMAP, IMAPS, POP, SMTP и AUTODISCOVER для автоматичекого обнаружения протоколов. Создать файл autodiscover.xml.

Остальные настройки находятся в файлах.

- Настройка в файле Autodiscover.md
- Настройка SRV записей.md
- Настройка почтовых протоколов.md
- Настройка API.md
- Установка Органайзера.md
- Установка модуля графики.md

