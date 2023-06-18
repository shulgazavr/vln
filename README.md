# Сетевые пакеты. VLAN. LACP.

### Описание/Пошаговая инструкция выполнения домашнего задания:
В Office1 в тестовой подсети появляется сервера с доп интерфесами и адресами в internal сети testLAN: </br>
C1 - 10.10.10.2</br>
C2 - 10.10.10.2</br>
S1- 10.10.10.254</br>
S2- 10.10.10.254</br>

Развести вланами:</br>
C1 <-> S1</br>
C2 <-> S2</br>

Между CR и IR "пробросить" 2 линка (общая inernal сеть) и объединить их в бонд. Проверить работу c отключением интерфейсов.</br>

### Карта сети:
![111](https://github.com/shulgazavr/vln/assets/105816449/f36e8180-3231-4b77-a375-ab6432bd9e05)

### Описание реализации:

1.
S1 и C1 соединены с R1 при помощи технологии vlan с vlanid 101.</br>
```
[vagrant@R1 ~]$ ip a
...
5: eth2.101@eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:d4:29:2b brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.1/24 brd 10.10.10.255 scope global eth2.101
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed4:292b/64 scope link 
       valid_lft forever preferred_lft forever
```
S2 и C2 соединены с R2 при помощи технологии vlan с vlanid 102.
```
[vagrant@S2 ~]$ ip a
...
4: eth1.102@eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:ac:dd:9f brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.254/24 brd 10.10.10.255 scope global eth1.102
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feac:dd9f/64 scope link 
       valid_lft forever preferred_lft forever
```

R1 и R2, в цепочках `POSTROUTING` таблицах `nat` имееют правила маскарадинга. 
```
[vagrant@R1 ~]$ sudo iptables -L -n -v -t nat
...       
Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  129  9812 MASQUERADE  all  --  *      !eth2   0.0.0.0/0            0.0.0.0/0 
  ```
2.
R1 и R2 соединены с CR "обычным" линком.
```
[vagrant@CR ~]$ ip a
...
6: eth4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:44:f2:dc brd ff:ff:ff:ff:ff:ff
    inet 192.168.55.1/24 brd 192.168.55.255 scope global noprefixroute eth4
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe44:f2dc/64 scope link 
       valid_lft forever preferred_lft forever
```
CR в цепочке `POSTROUTING` таблицы `nat` имеет правило маскарадинга.
```
[vagrant@CR ~]$ sudo iptables -L -n -v -t nat
...
Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  474 36080 MASQUERADE  all  --  *      bond0   0.0.0.0/0           !192.168.0.0/16 
```
3.
CR и IR соединены при помощи бондинга.
```
[vagrant@IR ~]$ ip a
...
3: eth1: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 9000 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 08:00:27:de:67:f6 brd ff:ff:ff:ff:ff:ff
4: eth2: <BROADCAST,MULTICAST,SLAVE,UP,LOWER_UP> mtu 9000 qdisc pfifo_fast master bond0 state UP group default qlen 1000
    link/ether 08:00:27:de:67:f6 brd ff:ff:ff:ff:ff:ff
5: bond0: <BROADCAST,MULTICAST,MASTER,UP,LOWER_UP> mtu 9000 qdisc noqueue state UP group default qlen 1000
    link/ether 08:00:27:de:67:f6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.1/24 brd 192.168.100.255 scope global bond0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fede:67f6/64 scope link 
       valid_lft forever preferred_lft forever
```
IR в цепочке `POSTROUTING` таблицы `nat` имеет правило маскарадинга.
```
[vagrant@IR ~]$ sudo iptables -L -n -v -t nat
...
Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
  645 49580 MASQUERADE  all  --  *      eth0    0.0.0.0/0           !192.168.0.0/16
```

### Проверка работы бондинга при отключении одного из интерфейсов.
Статус бондинга на IR:
```
[root@IR ~]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup) (fail_over_mac active)
Primary Slave: None
Currently Active Slave: eth1
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:12:00:42
Slave queue ID: 0

Slave Interface: eth2
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 08:00:27:a3:e4:22
Slave queue ID: 0
```

Если запустить `ping -f` на CR, а на IR в это же время отключить линк одно из интерфейсов бондинга (eth1), то можно увидеть статистику потери пакетов:
```
[root@CR ~]# ping -f 192.168.100.1
PING 192.168.100.1 (192.168.100.1) 56(84) bytes of data.
..^C
--- 192.168.100.1 ping statistics ---
72879 packets transmitted, 72877 received, 0% packet loss, time 9971ms
rtt min/avg/max/mdev = 0.036/0.097/20.063/0.156 ms, pipe 2, ipg/ewma 0.136/0.096 ms
```
