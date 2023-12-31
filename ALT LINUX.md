# ALT - LINUX    

## Задание 1.1  
__Цель задания:__  
Выполнить базовую настройку всех устройств:  
    a.Собрать топологию согласно рисунку. Все устройства работают на OC Linux - ALT  
            - ISP - Альт Сервер 10.2 (CLI)  
            - CLI - Альт Рабочая станция 10.2 (GUI)  
            - HQ-R - Альт Сервер 10.2 (CLI)  
            - HQ-SRV - Альт Сервер 10.2 (GUI)  
            - BR-R - Альт Сервер 10.2 (CLI)  
            - BR-SRV - Альт Сервер 10.2 (CLI)  
    b.Присвоить имена в соответствии с топологией  
    c.Рассчитать IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактировать таблицу.  
    d.Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустить этот пункт.  
    e.Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустить этот пункт.  

### Топология сети
![ALT топология](https://github.com/Clover136/demo2024/assets/148867684/d9b1435a-1f9c-4ea9-9b88-3bd6a27ea0bd)

### Таблица сети (разбитая на подсети)
|Имя устройства|Интерфейсы|  IPv4/IPv6  |Маска/Префикс|     Шлюз    |
|--------------|----------|-------------|-------------|-------------|
| ISP          | ens192   | 192.168.0.170 | /30         |           |
|              | ens224   | 192.168.0.162 | /30         |           |
|              | ens256   | 192.168.0.166 | /30         |           |
|              | ens161   | 10.12.13.88 | /24         | 10.12.13.254 |
| HQ-R         | ens192   | 192.168.0.2     | /25         |         |
|              | ens224   | 192.168.0.161 | /30         |  192.168.0.162 | 
| BR-R         | ens192   | 192.168.0.130 | /27         |             |
|              | ens224   | 192.168.0.165     | /30         | 192.168.0.166 |
| HQ-SRV       | ens192   | 192.168.0.1 | /25         | 192.168.0.2 |
| BR-SRV       | ens192   | 192.168.0.129| /27         | 192.168.0.130 |
| CLI          | ens192   | 192.168.0.169     | /30         | 192.168.0.170 |
---
### Выполнение задания  
### Метод, если интерфейсы были добалены до полной установки системы
Для начала узнал, какие интерфейсы есть на `ISP`:
```
ip a
```
После этого приступил к настройке статической маршрутизации  
Открыл файл options для нужного интерфейса:  
```
vim /etc/net/ifaces/ens192/options
```
Там поменял все как на примере:   
```
BOOTPROTO=static
TYPE=eth
CONFIG_WIRELESS=no
SYSTEMD_BOOTPROTO=static
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=no
```
После этого задал нужный адрес на интрефейс:  
```
echo 192.168.0.170/30 > /etc/net/ifaces/ens192/ipv4address
```
Если нужно добавить шлюз по умолчанию, то нужна эта команда:  
```
echo default via xxx.xxx.xxx.xxx > /etc/net/ifaces/xxx/ipv4route
```
`Вместо x, нужно вставить IP-адрес и номер интерфейса`  
Если нужно указать информацию о DNS-сервере, прописываем команду:  
```
echo nameserver 8.8.8.8 > /etc/resolv.conf
```
После этого перезагружаем сетевую службу:  
```
service network restart
```
И смотрим результат:  
```
ip a
```
Если на интерфейсе показывается 2 IP-адреса то  нужно отключить NetworkManager командой:
```
systemctl disable network.service NetworkManager
```
---
### Метод, если интерфейсы были добавлены после установки системы  
Для начала создаем папку с нужным интерфейсом по этому пути:  
```
mkdir /etc/net/ifaces/xxx
```
`Вместо x пишется нужный интерфейс`  
Далее в данной папку нужно создать файл `options` со следующими параметрами:
```
BOOTPROTO=static
TYPE=eth
CONFIG_WIRELESS=no
SYSTEMD_BOOTPROTO=dhcp4
CONFIG_IPV4=yes
DISABLED=no
NM_CONTROLLED=no
SYSTEMD_CONTROLLED=no
```
После этого задается нужный адрес на интрефейс:  
```
echo 192.168.0.170/30 > /etc/net/ifaces/ens192/ipv4address
```
Если нужно добавить шлюз по умолчанию, то нужна эта команда:  
```
echo default via xxx.xxx.xxx.xxx > /etc/net/ifaces/xxx/ipv4route
```
`Вместо x, нужно вставить IP-адрес и номер интерфейса`  
Если нужно указать информацию о DNS-сервере, прописываем команду:  
```
echo nameserver 8.8.8.8 > /etc/resolv.conf
```
После этого перезагружаем сетевую службу:  
```
service network restart
```
И смотрим результат:  
```
ip a
```
Если на интерфейсе показывается 2 IP-адреса то  нужно отключить NetworkManager командой:
```
systemctl disable network.service NetworkManager
```
---
### Все тоже самое повторил на других интерфейсах

---

## Настройка ТУННЕЛЯ между HQ-R и BR-R
Для начала нужно создать директорию для туннельного интерфейса:  
```
mkdir /etc/net/ifaces/tun1
```
Затем заполнить создать и заполнить файл `options` у туннеля:  
```
vim /etc/net/ifaces/tun1/options
```
```
TYPE=iptun
TUNTYPE=gre
TUNLOCAL=xxx.xxx.xxx.xxx
TUNREMOTE=xxx.xxx.xxx.xxx
TUNOPTIONS='ttl 64'
HOST=ens???
```
> в `TUNLOCAL` нужно вписать адрес смотрящий на ISP  
> в `TUNREMOTE` нужно вписать адрес устйства, к которому идет туннель  
> в `HOST` нужног вписать интерфейс смотрящий на ISP  

Дальше добавляем IP-адрес на туннель:
```
echo xxx.xxx.xxx.xxx/xx > /etc/net/ifaces/tun1/ipv4address
```
В конце перезагружаем службу network:  
```
systemctl restart network
```
**Все тоже самое делаем и на другом устройстве туннеля**

---

## Настройка динамической маршрутизации FRR
Для начала установим пакеты:  
```
apt-get update
```
Затем установим и сам FRR:  
```
apt-get -y install frr
```
И включим автозагрузку FRR:  
```
systemctl enable --now frr
```
Далее включаем демона:  
```
nano /etc/frr/daemons
```
И меняем `ospfd=no`  
На `ospfd =yes`  

Затем заходим в среду роутера:  
```
vtysh
```
И прописываем:  
```
conf t
    router ospf
    net 192.168.0.160/30 area 0
    net 192.168.0.164/30 area 0
    exit
ip forwarding
do w
```    
Иногда настройки vtysh слетают, для этого заходим в:
```
nano /etc/frr/frr.conf
```
И добавляем после `ipv6 forwarding` такую строчку:
```
ip forwarding
```

**Все тоже самое проделываю на HQ-R и BR-R**

---













## Настройка NAT через Firewalld
В настроках `options` у интерфейсов, должны быть такие значения:  
```
NM_CONTROLLED=no
DISABLED=no
```
Так же стоит отключить NetworkManager, если ранее это не было сделанно:  
```
systemctl disable network.service NetworkManager
```
Далее нужно установить firewalld:  
```
apt-get -y install firewalld
```
После этого включить автозагрузку:  
```
systemctl enable --now firewalld
```
Добавляем правила к исходящим пакетам:
```
firewall-cmd --permanent --zone=public --add-interface=ens???
```
> zone=public - интерфейс смотрящий в сторону Интернета  

Добавляем правила к входящим пакетам:  
```
firewall-cmd --permanent --zone=trusted --add-interface=ens???
```
> zone=trusted - доверенные адреса внутренней локальной сети  

Включаем NAT:  
```
firewall-cmd --permanent --zone=public --add-masquerade
```
Сохранение правил:  
```
firewall-cmd --reload
```

### Дальнейшая настройка firewalld с ISP
Далее нужно включить пересылку пакетов с BR-R на HQ-R и наоборот:  
```
firewall-cmd --direct --permanent --add-rule ipv4 filter FORWARD 0 -i ens224 -o ens256 -j ACCEPT  
firewall-cmd --direct --permanent --add-rule ipv4 filter FORWARD 0 -i ens256 -o ens224 -j ACCEPT  
```
Затем открывем порты OSPF:  
```
firewall-cmd --permanent --zone=trusted --add-port=89/tcp  
firewall-cmd --permanent --zone=trusted --add-port=89/udp  
```
### Настройка firewalld на HQ-R, BR-R
HQ-R и BR-R Включаем пересылку между интерфейсом смотрящим в ISP и туннелем:  
```
firewall-cmd --direct --permanent --add-rule ipv4 filter FORWARD 0 -i ens224 -o tun1 -j ACCEPT  
firewall-cmd --direct --permanent --add-rule ipv4 filter FORWARD 0 -i tun1 -o ens224 -j ACCEPT
```
Открываем порты OSPF на HQ-R и BR-R:  
```
firewall-cmd --permanent --zone=trusted --add-port=89/tcp  
firewall-cmd --permanent --zone=trusted --add-port=89/udp  
```
А так же нужно не забыть добавить туннель в зону `trusted` на HQ-R и BR-R:  
```
firewall-cmd --permanent --zone=trusted --add-interface=tun1
```
**На этом настройка закончена**

> Если Firewalld после перезагрузки машины не загружает введёные команды автоматически, а только после команды:
> ```
> firewall-cmd --reload
> ```
> То стоит проверить `options` у интерфейсов, и при необходимости выключить NetworkManager:
> ```
> NM_CONTROLLED=no
> ```
> Так же может помочь команда:
> ```
> systemctl disable network.service NetworkManager
> ```

---








###
