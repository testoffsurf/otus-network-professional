# Масштабируемость и дизайн iBGP

## Цель:
Настроить iBGP в офисе Москва
Настроить iBGP в сети провайдера Триада
Организовать полную IP связанность всех сетей

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
R14(config-router)#exit

R15(config)#router bgp 1001
R15(config-router)#neighbor 10.77.0.254 remote-as 1001
R15(config-router)#neighbor 10.77.0.254 update-source Loopback0
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










### Настройте офис Москва так, чтобы приоритетным провайдером стал Ламас





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
