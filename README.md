# **Введение**

В данном домашнем задании нам необходимо получить практический опыт работы c PXE (Preboot eXecution Environment).

---

Для выполнения данного ДЗ нам необходимо настроить сервер PXE, на котором поднять Web-сервер Apache, TFTP-сервер и DHCP-сервер. Вторая виртуальная машина будет выступать в роли pxe-клиента. Все настройки сделаны согласно методички.

- Для запуска выполняем следующее:
```
git clone https://github.com/Dmitriy-Iv/Otus-Pr-linux-HW19.git
cd Otus-Pr-linux-HW19/
vagrant up
```

После запуска `vagrant up` - произойдёт настройка PXE-сервера через ansible playbook, и когда создастся вторая виртуальная машина - pxe-клиент и мы увидим на экране, что vagrant не может подключится к ней, можно отменить дальнейшее выполнение через `Ctrl+C`. Дальше перейти в Virtual box и открыть консоль этой машины, где мы видим, что начнётся актоматическая установка нашего скачанного образа - CentOS-8.4 в автоматическом режиме, потому что мы заранее подготовили файл kickstart (файл ответов на вопросы при установке - выбор языка, разметка диска и т.д.).

- Скриншоты:
Меню загрузки:
![alt text](/screenshots/hw19-1.PNG?raw=true "Screenshot1")

Установка ОС CentOS-8.4:
![alt text](/screenshots/hw19-2.PNG?raw=true "Screenshot2")

![alt text](/screenshots/hw19-3.PNG?raw=true "Screenshot3")

После установки, выключаем машину, меняем в Virtual Box в настроках машины загрузку с Network на Hard Disk и стартуем. Логинимся под root и смотрим версию OC.
![alt text](/screenshots/hw19-4.PNG?raw=true "Screenshot4")