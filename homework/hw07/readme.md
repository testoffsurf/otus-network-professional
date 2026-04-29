# Масштабируемость и дизайн iBGP

## Цель:
Настроить iBGP в офисе Москва <br>
Настроить iBGP в сети провайдера Триада <br>
Организовать полную IP связанность всех сетей <br>

## Задание:
  1. Настроите iBGP в офисом Москва между маршрутизаторами R14 и R15.
  2. Настроите iBGP в провайдере Триада, с использованием RR.
  3. Настройте офиса Москва так, чтобы приоритетным провайдером стал Ламас.
  4. Настройте офиса С.-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно.
  5. Все сети в лабораторной работе должны иметь IP связность.

### Топология
<center><img src="bgp_topologi.png" align="middle"></center>

<br>

### Настроите iBGP в офисом Москва между маршрутизаторами R14 и R15
Прямого соединения между маршрутизаторами R14 и R15 у нас нет, но есть соединения через маршрутизаторы R12 и R13 на которых работает алгоритм маршрутизации OSPF и следовательно есть маршруты до Loopback-интерфейсов. iBGP соседство можно построить с помощью Loopback-интерфейсов, для этого сделаем следующие изменения на маршрутизаторах R14 и R15:
```
R14(config)#router bgp 1001
R14(config-router)#neighbor 10.77.0.253 remote-as 1001
R14(config-router)#neighbor 10.77.0.253 update-source Loopback0
R14(config-router)#neighbor 10.77.0.253 next-hop-self
R14(config-router)#exit

R15(config)#router bgp 1001
R15(config-router)#neighbor 10.77.0.254 remote-as 1001
R15(config-router)#neighbor 10.77.0.254 update-source Loopback0
R15(config-router)#neighbor 10.77.0.254 next-hop-self
R15(config-router)#exit
```

Воспользуемся командой <b>show ip bgp summary</b> чтобы посмотреть краткую информацию о состоянии BGP-соединении на маршрутизаторах R14 и R15:

</code></pre>
</details>
<details>
<summary>R14</summary>
<pre><code>
R14#sh ip bgp summary
BGP router identifier 10.77.0.254, local AS number 1001
BGP table version is 7, main routing table version 7
5 network entries using 700 bytes of memory
8 path entries using 640 bytes of memory
5/4 BGP path/bestpath attribute entries using 720 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2132 total bytes of memory
BGP activity 5/0 prefixes, 8/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.77.0.253     4         1001      25      25        7    0    0 00:19:52        3
100.78.0.1      4          101      23      25        7    0    0 00:16:31        4
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>R15</summary>
<pre><code>
R15#sh ip bgp summary
BGP router identifier 10.77.0.253, local AS number 1001
BGP table version is 6, main routing table version 6
5 network entries using 700 bytes of memory
8 path entries using 640 bytes of memory
5/4 BGP path/bestpath attribute entries using 720 bytes of memory
3 BGP AS-PATH entries using 72 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 2132 total bytes of memory
BGP activity 5/0 prefixes, 8/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.77.0.254     4         1001      27      27        6    0    0 00:20:59        3
100.77.0.5      4          301      24      26        6    0    0 00:17:53        4
</code></pre>
</details>

Мы видим что у маршрутизатора R15 два соседа и один из них в автономной системе 1001 с IP-адресом 10.77.0.254.

### Настроите iBGP в провайдере Триада, с использованием RR
Определим маршрутизаторы R23 и R24 как маршрутизаторы с технологией "Route Reflector". Для того чтобы технология "Route Reflector" начала работать на данных маршрутизаторах настроим ее следующими командами:
```
R23(config)#router bgp 520
R23(config-router)#neighbor 100.0.0.252 remote-as 520
R23(config-router)#neighbor 100.0.0.252 update-source Loopback0
R23(config-router)#neighbor 100.0.0.252 next-hop-self
R23(config-router)#neighbor 100.0.0.253 remote-as 520
R23(config-router)#neighbor 100.0.0.253 update-source Loopback0
R23(config-router)#neighbor 100.0.0.253 route-reflector-client
R23(config-router)#neighbor 100.0.0.253 next-hop-self
R23(config-router)#neighbor 100.0.0.254 remote-as 520
R23(config-router)#neighbor 100.0.0.254 update-source Loopback0
R23(config-router)#neighbor 100.0.0.254 route-reflector-client
R23(config-router)#neighbor 100.0.0.254 next-hop-self
R23(config-router)#exit

R24(config)#router bgp 520
R24(config-router)#neighbor 100.0.0.251 remote-as 520
R24(config-router)#neighbor 100.0.0.251 update-source Loopback0
R24(config-router)#neighbor 100.0.0.251 next-hop-self
R24(config-router)#neighbor 100.0.0.253 remote-as 520
R24(config-router)#neighbor 100.0.0.253 update-source Loopback0
R24(config-router)#neighbor 100.0.0.253 route-reflector-client
R24(config-router)#neighbor 100.0.0.253 next-hop-self
R24(config-router)#neighbor 100.0.0.254 remote-as 520
R24(config-router)#neighbor 100.0.0.254 update-source Loopback0
R24(config-router)#neighbor 100.0.0.254 route-reflector-client
R24(config-router)#neighbor 100.0.0.254 next-hop-self
R24(config-router)#exit
```

Воспользуемся командой <b>show ip bgp summary</b> чтобы посмотреть краткую информацию о состоянии BGP-соединении на маршрутизаторах R23 и R24:

</code></pre>
</details>
<details>
<summary>R23</summary>
<pre><code>
R23#sh ip bgp summary
BGP router identifier 100.0.0.251, local AS number 520
BGP table version is 20, main routing table version 20
6 network entries using 840 bytes of memory
7 path entries using 560 bytes of memory
3/3 BGP path/bestpath attribute entries using 432 bytes of memory
1 BGP rrinfo entries using 24 bytes of memory
1 BGP AS-PATH entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1880 total bytes of memory
BGP activity 8/2 prefixes, 14/7 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
100.0.0.2       4          101       2       7       20    0    0 00:00:07        0
100.0.0.252     4          520      32      29       20    0    0 00:20:43        5
100.0.0.253     4          520      25      27       20    0    0 00:17:28        0
100.0.0.254     4          520      27      31       20    0    0 00:18:45        1
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>R24</summary>
<pre><code>
R24#sh ip bgp summary
BGP router identifier 100.0.0.252, local AS number 520
BGP table version is 16, main routing table version 16
7 network entries using 980 bytes of memory
13 path entries using 1040 bytes of memory
6/4 BGP path/bestpath attribute entries using 864 bytes of memory
1 BGP rrinfo entries using 24 bytes of memory
4 BGP AS-PATH entries using 96 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 3004 total bytes of memory
BGP activity 9/2 prefixes, 17/4 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
100.0.0.6       4          301      10      13       16    0    0 00:05:24        4
100.0.0.10      4         2042      13      14       16    0    0 00:04:41        2
100.0.0.251     4          520      31      34       16    0    0 00:22:06        4
100.0.0.253     4          520      65      76       16    0    0 00:55:03        0
100.0.0.254     4          520      62      67       16    0    0 00:50:19        1
</code></pre>
</details>

Мы видим что у маршрутизатора R24 пять соседей: один сосед из автономной системы 301, один сосед из автономной системы 2042 и три соседа из автономной системы 520.

### Настройте офис Москва так, чтобы приоритетным провайдером стал Ламас
Для того чтобы в офисе Москва приоритетным провайдером стал Ламас, необходимо использовать один из атрибутов BGP, а именно Local Preference. На маршрутизаторе R15 напишем соответствующий <b>route-map</b>, меняющий значение Local Preference со 100 на 150. После чего применим <b>route-map</b> на входящие маршруты от интернет провайдера Ламас:
```
R15(config)#route-map SET-LOCAL-PREFERENCE permit 10
R15(config-route-map)#set local-preference 150
R15(config-route-map)#exit
R15(config)#
R15(config)#router bgp 1001
R15(config-router)#neighbor 100.77.0.5 route-map SET-LOCAL-PREFERENCE in
R15(config-router)#exit
```








### Настройте офис Санкт-Петербург так, чтобы трафик до любого офиса распределялся по двум линкам одновременно
Для того чтобы трафик с офиса Санкт-Петербург распределялся по двум линкам одновременно воспользуемся следующей командой для настройки BGP:

```
R18(config)#router bgp 2042
R18(config-router)#bgp bestpath as-path multipath-relax
R18(config-router)#maximum-paths 2
R18(config-router)#exit
```

<br>

Полные файлы изменений приведены [здесь](config/)
