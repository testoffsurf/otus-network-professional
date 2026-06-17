# DMVPN

## Цель:
Настроить GRE между офисами Москва и Санкт-Петербург <br>
Настроить DMVPN между офисами Москва и Чокурдах, Лабытнанги <br>

## Задание:
  1. Настроите GRE между офисами Москва и Санкт-Петербург
  2. Настроите DMVPN Ммжду Москва и Чокурдах, Лабытнанги

### Топология
<center><img src="dmvpn_topologi.png" align="middle"></center>


### Настроите GRE между офисами Москва и Санкт-Петербург
Для организации GRE-туннеля между офисом в г. Москва и офисом в г. Санкт-Петербург воспользуемся следующими конфигурационными командами на соответствующих маршрутизаторах в этих городах.

```
R15(config)#interface Tunnel0
R15(config-if)#ip address 172.16.0.1 255.255.255.252
R15(config-if)#ip mtu 1400
R15(config-if)#ip tcp adjust-mss 1360
R15(config-if)#tunnel source Ethernet0/2
R15(config-if)#tunnel destination 100.0.0.10
R15(config-if)#tunnel path-mtu-discovery
R15(config-if)#exit
```

```
R18(config)#interface Tunnel0
R18(config-if)#ip address 172.16.0.2 255.255.255.252
R18(config-if)#ip mtu 1400
R18(config-if)#ip tcp adjust-mss 1360
R18(config-if)#tunnel source Ethernet0/2
R18(config-if)#tunnel destination 100.77.0.6
R18(config-if)#tunnel path-mtu-discovery
R18(config-if)#exit
```







<br>

Полные файлы изменений приведены [здесь](config/)



