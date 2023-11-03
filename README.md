# DEMO2024 Galiev

## Задание 1.1
__Цель задания:__
1. Выполнить базовую настройку всех устройств:
    - Собрать топологию согласно рисунку. Все устройства работают на OC Linux - Debian  
    - Присвоить имена в соответствии с топологией  
    - Рассчитать IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактировать таблицу.  
    - Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустить этот пункт.  
    - Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустить этот пункт.  

### Топология сети
![Топология](https://github.com/Clover136/demo2024/assets/148867684/2f2aac75-97a7-4aad-b53f-5aa482b7409c)


### Таблица сети (разбитая на подсети)
|Имя устройства|Интерфейсы|  IPv4/IPv6  |Маска/Префикс|     Шлюз    |
|--------------|----------|-------------|-------------|-------------|
| ISP          | ens192   | 192.168.0.162 | /30         |             |
|              | ens224   | 192.168.0.166 | /30         |             |
|              | ens256   | 10.12.13.88 | /24         |             |
| HQ-R         | ens192   | 192.168.0.161     | /30         | 192.168.0.162     |
|              | ens224   | 192.168.0.2 | /25         |             | 
| BR-R         | ens192   | 192.168.0.130 | /27         |             |
|              | ens224   | 192.168.0.165     | /30         | 192.168.0.166 |
| HQ-SRV       | ens192   | 192.168.0.1 | /25         | 192.168.0.2 |
| BR-SRV       | ens192   | 192.168.0.129| /27         | 192.168.0.130 |
---
## Выполнение задания 1.1
Для начала прописал команду:
```
ip a
```   
Она для нужна, что бы узнать интерфейсы нашей машины

Затем зашел в конфигурационный файл наших интерфейсов:
```
nano /etc/network/interfaces
```
После чего изменил их согласно топологии и таблице адресации, как показано ниже:


 >#The primary network interface
 >
 >auto ens192  
 >iface ens192 inet static  
 >address 192.168.0.162  
 >netmask 255.255.255.252  
 >
 >auto ens256  
 >iface ens256 inet static  
 >address 10.12.13.88  
 >gateway 10.10.200.200  
 >netmask 255.255.255.0   
 >
 >auto ens224  
 >iface ens224 inet static  
 >address 192.168.0.166  
 >netmask 255.255.255.252  


Затем я сохранил конфигурацию:  
```
ctrl+s
```

И вышел  из неё: 
```
ctrl+x
```

После чего перезагрузил сетевой сервис: 
```
systemctl restart networking
```

Снова пишу команду: `ip a`  
Вот какой должен выйти итог:
```
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:79:0f:40 brd ff:ff:ff:ff:ff:ff
    altname enp11s0
    inet 192.168.0.162/30 brd 192.168.0.163 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe79:f40/64 scope link
       valid_lft forever preferred_lft forever
3: ens224: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:79:0f:4a brd ff:ff:ff:ff:ff:ff
    altname enp19s0
    inet 192.168.0.166/30 brd 192.168.0.167 scope global ens224
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe79:f4a/64 scope link
       valid_lft forever preferred_lft forever
4: ens256: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:79:0f:54 brd ff:ff:ff:ff:ff:ff
    altname enp27s0
    inet 10.12.13.88/24 brd 10.12.13.255 scope global ens256
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe79:f54/64 scope link
       valid_lft forever preferred_lft forever
```

**Всю ту же работу я проделал и на других машинах**
---


## Настройка NAT
На устройстве **ISP** прописываем команду:  
```
apt install iptables
```
Далее зашел в конфигурцию файл:  
```
nano /etc/sysctl.conf
```
После чего убрал убрал знак решетки `(#)`, с этой строки:  
```
#net.ipv4.ip_forward=1
```
И проверил выполнение командой: `sysctl -p`  
Затем прописываем команду:  
```
ip tables -A POSTROUTING -t nat -j MASQUERADE
```
Затем создаем файл для автоматического запуска NAT псоле перезагрузки устройства:  
```
nano /etc/network/if-pre-up.d/nat
```
И вписываем в файл это:  
>#!/bin/sh  
>/sbin/iptables -A POSTROUTING -t nat -j MASQUERADE

В конце даем права для файла:  
```
chmod +x /etc/network/if-pre-up.d/nat
```

**Всю ту же работу выполняем на устройствах BR-R и HQ-R**  
---

## Выполнение задания 1.2
__Цель задания:__  
Настройка внутренней динамической маршрутизации по средствам FRR  

Сначало на устройстве **ISP** устанавливаем  frr  
```
apt install frr
```
Далее открыаем файл:  
```
nano /etc/frr/daemons
```
И затем меняем значения:  
Вместо `ospfd=no`  
На `ospfd=yes`  
Перезагружаем службу FRR:
```
systemctl restart frr
```
Переходим в среду роутера, командой:
```
vtysh
```
Затем настраиваем ospf, прописывая адреса ближайших сетевых устройств (BR-R, HQ-R):  
>conf t  
>router ospf  
>    net 192.168.0.160/30 area 0  
>    net 192.168.0.164/30 area 0  
>sh ip ospf neighbor  

**Далее настраиваем FRR на HQ-R и BR-R**  

В конце проверяем правильность настройки пропинговав:  
`HQ-SRV` с `BR-SRV`  
`BR-SRV` с `HQ-SRV`  

---

