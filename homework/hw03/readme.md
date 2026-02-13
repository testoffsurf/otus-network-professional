# OSPF. Фильтрация

## Цель:
Настроить OSPF офисе Москва. Разделить сеть на зоны. Настроить фильтрацию между зонами.

## Задание:
  1. Маршрутизаторы R14-R15 должны находиться в зоне 0 - backbone
  2. Маршрутизаторы R12-R13 находяться в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию
  3. Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию
  4. Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101




### Маршрутизаторы R14-R15 должны находиться в зоне 0 - backbone
На интерфейсах маршрутизаторов R14, R15, R12 и R13 смотрящих в зону 0 необходимо сконфигурировать интерфейсы следующим образом:

```
interface Ethernet X/X
 ip ospf 1 area 0
```

А также не забыть запустить процесс OSPF на всех вышеуказанных маршрутизаторах, прописать необходимые подсети, loopback-интерфейсы и указать ROUTER-ID (пример для маршрутизатора R14):

```
router ospf 1
 router-id 10.77.0.254
 network 10.77.0.0 0.0.0.3 area 101
 network 10.77.0.4 0.0.0.3 area 0
 network 10.77.0.8 0.0.0.3 area 0
 network 10.77.0.254 0.0.0.0 area 0
```


### Маршрутизаторы R12-R13 должны находиться в зоне 10. Дополнительно к маршрутам должны получать маршрут по умолчанию
На интерфейсах маршрутизаторов R12, R13 и L3-коммутаторов SW4, SW5 смотрящих в зону 10 необходимо сконфигурировать интерфейсы следующим образом:

```
interface Ethernet X/X
 ip ospf 1 area 10
```

А также не забыть запустить процесс OSPF на всех вышеуказанных маршрутизаторах и L3-коммутаторах, прописать необходимые подсети, loopback-интерфейсы и указать ROUTER-ID (пример для маршрутизатора R12):

```
router ospf 1
 router-id 10.77.0.251
 network 10.77.0.24 0.0.0.3 area 10
 network 10.77.0.28 0.0.0.3 area 10
```

На маршрутизаторе R14 анонсируем маршрут по умолчанию:
```
router ospf 1
 default-information originate always
```

Командой <b>show ip route</b> посмотрим, присутствует ли маршрут по умолчанию в зоне 10:

</code></pre>
</details>
<details>
<summary>Area 10</summary>
<pre><code>
R13(config)#do sh ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.77.0.9 to network 0.0.0.0

O*E2  0.0.0.0/0 [110/1] via 10.77.0.9, 00:32:52, Ethernet0/3
      10.0.0.0/8 is variably subnetted, 18 subnets, 2 masks
O IA     10.77.0.0/30 [110/20] via 10.77.0.9, 00:34:33, Ethernet0/3
O        10.77.0.4/30 [110/20] via 10.77.0.9, 00:34:33, Ethernet0/3
C        10.77.0.8/30 is directly connected, Ethernet0/3
L        10.77.0.10/32 is directly connected, Ethernet0/3
O        10.77.0.12/30 [110/20] via 10.77.0.17, 00:34:33, Ethernet0/2
C        10.77.0.16/30 is directly connected, Ethernet0/2
L        10.77.0.18/32 is directly connected, Ethernet0/2
O IA     10.77.0.20/30 [110/20] via 10.77.0.17, 00:34:33, Ethernet0/2
O        10.77.0.24/30 [110/20] via 10.77.0.34, 00:29:57, Ethernet0/1
</code></pre>
</details>

Маршрут E*O2 0.0.0.0/0 присутствует в таблице маршрутизации.


### Маршрутизатор R19 находится в зоне 101 и получает только маршрут по умолчанию
Интерфейс <b>e0/0</b> маршрутизатора R19 сконфигурируем следующим образом:

```
interface Ethernet0/0
 ip ospf 1 area 101
```

Запускаем OSPF процесс на маршрутизаторе R19 и объявляем зону 101 как Totally stub Area.
<pre><code>
router ospf 1
 router-id 10.77.0.252
 network 10.77.0.0 0.0.0.3 area 101
 <b>area 101 stub no-summary</b>
</code></pre>

Также со стороны маршрутизатора R14 зону 101 объявляем как Totally stub Area, тем самым предотвращаем обмен LSA 3 типа. Таблица маршрутизации на R19 имеет следующий вид:

</code></pre>
</details>
<details>
<summary>R19</summary>
<pre><code>
R19#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is 10.77.0.1 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 10.77.0.1, 01:21:11, Ethernet0/0
</code></pre>
</details>

Мы видим маршрут как Inter Area. Маршрут остался один, он же Last Resort Gateway.


### Маршрутизатор R20 находится в зоне 102 и получает все маршруты, кроме маршрутов до сетей зоны 101























Полные файлы изменений приведены [здесь](config/)
