# **Введение**

В данном домашнем задании нам необходимо получить практический опыт работы c PXE (Preboot eXecution Environment).

---

Для выполнения данного ДЗ нам необходимо настроить сервер PXE, на котором поднять Web-сервер Apache, TFTP-сервер и DHCP-сервер. Вторая виртуальная машина будет выступать в роли pxe-клиента. Все настройки сделаны соогласно методички.

- Для запуска выполняем следующее:
```
git clone https://github.com/Dmitriy-Iv/Otus-Pr-linux-HW19.git
cd Otus-Pr-linux-HW19/
vagrant up
```

После запуска `vagrant up` - произойдёт настройка PXE-сервера через ansible playbook, и когда создасться вторая виртуальная машина - pxe-клиент и мы увидим на экране, что vagrant не может подключится к ней, можно отменить дальнейшее выполнение через `Ctrl+C`. Дальше перейти в Virtual box и открыть консоль этой машины, где мы видим, что начнётся актоматическая установка нашего скачанного образа - CentOS-8.4 в автоматическом режиме, потому что мы заранее подготовили файл kickstart (файл ответов на вопросы при установке - выбор языка, разметка диска и т.д.).

- Скриншоты:
Меню загрузки:
![alt text](/screenshots/hw19-1.PNG?raw=true "Screenshot1")

Установка ОС CentOS-8.4:
![alt text](/screenshots/hw19-2.PNG?raw=true "Screenshot2")

![alt text](/screenshots/hw19-3.PNG?raw=true "Screenshot3")

После установки, выключаем машину, меняем в Virtual Box в настроках машины загрузку с Network на Hard Disk и стартуем. Логинимся под root и смотрим версию OC.
![alt text](/screenshots/hw19-4.PNG?raw=true "Screenshot4")









































Для выполнения этого ДЗ выполняем запуск Vagrantfile, в котором создаются две Vm: client-borg и server-borg. С помощью ansible настраиваем создание хранилища для хранения бэкапов на server-borg в директории /var/backup. Далее инициализируем borg на backup сервере с client сервера (выполняется при помощи скрипта create_repo.sh). После чего создаётся сервис borg-backup.service который в автоматическом режиме бэкапит директорию /etc.

- Для запуска выполняем следующее:
```
git clone https://github.com/Dmitriy-Iv/Otus-Pr-linux-HW18.git
cd Otus-Pr-linux-HW18/Ansible/
vagrant up
```

- Спустя пять-шесть минут заходим на client-borg и проверяем, появился ли у нас бэкап.
```
vagrant ssh client-borg
sudo -i
borg list borg@192.168.50.20:/var/backup/
(ввод пароля 'Otus1234')
```

- Что попало в архив можно посмотреть, открыв лог на client-borg `/var/log/borg.log`. Настроено через rsyslog (добавлены соотвествующие строки в borg-backup.service, создан файл конфигурации `/etc/rsyslog.d/borg_backup.conf`).
```
[root@client-borg ~]# cat /var/log/borg.log
Jun 16 15:34:46 client-borg borg_backup: ------------------------------------------------------------------------------
Jun 16 15:34:46 client-borg borg_backup: Archive name: etc-2022-06-16_15:34:43
Jun 16 15:34:46 client-borg borg_backup: Archive fingerprint: 8a0e520f9a69005b12c31c1cfb02bdf481dfab58fc640baad4780cdd6658c06e
Jun 16 15:34:46 client-borg borg_backup: Time (start): Thu, 2022-06-16 15:34:44
Jun 16 15:34:46 client-borg borg_backup: Time (end):   Thu, 2022-06-16 15:34:46
Jun 16 15:34:46 client-borg borg_backup: Duration: 1.69 seconds
Jun 16 15:34:46 client-borg borg_backup: Number of files: 1703
Jun 16 15:34:46 client-borg borg_backup: Utilization of max. archive size: 0%
Jun 16 15:34:46 client-borg borg_backup: ------------------------------------------------------------------------------
Jun 16 15:34:46 client-borg borg_backup: Original size      Compressed size    Deduplicated size
Jun 16 15:34:46 client-borg borg_backup: This archive:               28.44 MB             13.50 MB             11.85 MB
Jun 16 15:34:46 client-borg borg_backup: All archives:               28.44 MB             13.50 MB             11.85 MB
Jun 16 15:34:46 client-borg borg_backup: Unique chunks         Total chunks
Jun 16 15:34:46 client-borg borg_backup: Chunk index:                    1286                 1702

```

- Теперь попробуем восстановить что-нибудь из бэкапа. Останавливаем бэкап.
```
[root@client-borg ~]# systemctl stop borg-backup.timer
```

- Проверяем что находится в бэкапе. Для этого смотрим сначала название бэкапа (в нашем случае ниже это `etc-2022-06-19_15:21:16`), а потом выводим его содержимое.
```
[root@client-borg ~]# borg list borg@192.168.50.20:/var/backup/
Enter passphrase for key ssh://borg@192.168.50.20/var/backup:
etc-2022-06-19_15:21:16              Sun, 2022-06-19 15:21:17 [bd40d8633f2d30c0c10bb37596cee871024b39a8412f08749895bdd1af8c839c]

[root@client-borg ~]# borg list borg@192.168.50.20:/var/backup/::etc-2022-06-19_15:21:16 | head - 10
==> standard input <==
Enter passphrase for key ssh://borg@192.168.50.20/var/backup:
drwxr-xr-x root   root          0 Sun, 2022-06-19 15:21:01 etc
-rw------- root   root          0 Fri, 2020-05-01 01:04:55 etc/crypttab
lrwxrwxrwx root   root         17 Fri, 2020-05-01 01:04:55 etc/mtab -> /proc/self/mounts
-rw-r--r-- root   root         12 Sun, 2022-06-19 15:19:05 etc/hostname
-rw-r--r-- root   root       2388 Fri, 2020-05-01 01:08:36 etc/libuser.conf
-rw-r--r-- root   root       2043 Fri, 2020-05-01 01:08:36 etc/login.defs
-rw-r--r-- root   root         37 Fri, 2020-05-01 01:08:36 etc/vconsole.conf
-rw-r--r-- root   root         19 Fri, 2020-05-01 01:08:36 etc/locale.conf
-rw-r--r-- root   root        450 Sun, 2022-06-19 15:19:10 etc/fstab
-rw-r--r-- root   root       1186 Fri, 2020-05-01 01:08:37 etc/passwd
head: cannot open '10' for reading: No such file or directory

```

- Видим, что в бэкап попал весь каталог `/etc`. Достаём из архива его содержимое.
```
[root@client-borg ~]# borg extract borg@192.168.50.20:/var/backup/::etc-2022-06-19_15:21:16 etc/
Enter passphrase for key ssh://borg@192.168.50.20/var/backup:

[root@client-borg ~]# ls
anaconda-ks.cfg  create_repo.sh  etc  original-ks.cfg

[root@client-borg ~]# ls -l etc/ | wc -l
182
```

- Удаляем `/etc`.
```
[root@client-borg ~]# rm -rf /etc
rm: cannot remove '/etc': Device or resource busy

[root@client-borg ~]# ls /etc/ | wc -l
0


- Видим, что файлов в директории `/etc` не осталось, далее восстанавливаем из ранее скопированного архива файлы обратно.

[root@client-borg ~]# cp -Rf etc/* /etc/

[root@client-borg ~]# ls /etc | wc -l
181
```