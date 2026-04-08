# BGP. Продолжение

## Цель:
Настроить BGP между автономными системами. Организовать доступность между офисами Москва и С.-Петербург

## Задание:
  1. Настроите eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас
  2. Настроите eBGP между провайдерами Киторн и Ламас
  3. Настроите eBGP между Ламас и Триада
  4. Настроите eBGP между офисом С.-Петербург и провайдером Триада
  5. Организуете IP доступность между пограничным роутерами офисами Москва и С.-Петербург.


<br>

### Топология
<center><img src="bgp_extension.png" align="middle"></center>

<br>

### Настроите eBGP между офисом Москва и двумя провайдерами - Киторн и Ламас
Для того чтобы настроить eBGP между офисом Москва и двумя интернет провайдерами Киторн и Ламас необходимо сначала запустить процесс BGP на соответствующих устройствах (маршрутизаторы: R14, R15, R22, R21).

```
R14(config)#router bgp 1001
R14(config-router)#bgp log-neighbor-changes
R14(config-router)#neighbor 100.78.0.1 remote-as 101
R14(config-router)#exit

R15(config)#router bgp 1001
R15(config-router)#bgp log-neighbor-changes
R15(config-router)#neighbor 100.77.0.5 remote-as 301
R15(config-router)#exit

R22(config)#router bgp 101
R22(config-router)#bgp log-neighbor-changes
R22(config-router)#neighbor 100.78.0.2 remote-as 1001
R22(config-router)#exit

R21(config)#router bgp 301
R21(config-router)#bgp log-neighbor-changes
R21(config-router)#neighbor 100.77.0.6 remote-as 1001
R21(config-router)#exit
```

Командой <b>show ip bgp neighbors</b> посмотрим детальную информацию о BGP-соседях:
</code></pre>
</details>
<details>
<summary>show ip bgp neighbors</summary>
<pre><code>
R14#sh ip bgp neighbors
BGP neighbor is 100.78.0.1,  remote AS 101, external link
  BGP version 4, remote router ID 100.78.0.254
  BGP state = Established, up for 01:25:18
  Last read 00:00:16, last write 00:00:31, hold time is 180, keepalive interval is 60 seconds
  Neighbor sessions:
    1 active, is not multisession capable (disabled)
  Neighbor capabilities:
    Route refresh: advertised and received(new)
    Four-octets ASN Capability: advertised and received
    Address family IPv4 Unicast: advertised and received
    Enhanced Refresh Capability: advertised and received
    Multisession Capability:
    Stateful switchover support enabled: NO for session 1
</code></pre>
</details>

Мы видим что у маршрутизатора R14 один сосед с IP-адресом 100.78.0.1 в автономной системе 101. Состояние BGP-соседства – <b>ESTABLISHED</b>.

### Настроите eBGP между провайдерами Киторн и Ламас
Так как BGP процесс уже запушен на предыдущем шаге и для того чтобы настроить eBGP между двумя интернет провайдерами Киторн и Ламас, нам необходимо всего лишь в текущий BGP процесс на соответствующих устройствах (маршрутизаторы: R22, R21) дописать необходимые нам соседства.

```
R22(config)#router bgp 101
R22(config-router)#neighbor 100.77.0.1 remote-as 301
R22(config-router)#exit

R21(config)#router bgp 301
R21(config-router)#neighbor 100.77.0.2 remote-as 101
R21(config-router)#exit
```

Воспользуемся командой <b>show ip bgp summary</b> чтобы посмотреть краткую информацию о состоянии BGP-соединении на конкретном маршрутизаторе:
</code></pre>
</details>
<details>
<summary>show ip bgp summary</summary>
<pre><code>
R21#show ip bgp summary
BGP router identifier 100.77.0.254, local AS number 301
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
100.77.0.2      4          101       4       4        1    0    0 00:02:20        0
100.77.0.6      4         1001      15      16        1    0    0 00:10:41        0
</code></pre>
</details>

Мы видим что у маршрутизатора R21 два BGP-соседа: один сосед с IP-адресом 100.77.0.2 в автономной системе 101; второй сосед с IP-адресом 100.77.0.6 в автономной системе 1001. По состоянию колонки "State/PfxRcd" мы можем определить в каком состоянии находиться то или иное соседство.

</code></pre>
</details>
<details>
<summary>State/PfxRcd</summary>
<pre><code>
R21#sh ip bgp summary
BGP router identifier 100.77.0.254, local AS number 301
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
100.77.0.2      4          101      24      23        1    0    0 00:19:43        0
100.77.0.6      4         1001       0       0        1    0    0 00:01:23 Idle
</code></pre>
</details>

### Настроите eBGP между провайдерами Ламас и Триада
Для того чтобы настроить eBGP между провайдерами Ламас и Триада нам необходимо на соответствующих устройствах (маршрутизаторы: R24, R21) дописать необходимые нам соседства.

```
R24(config)#router bgp 520
R24(config-router)#neighbor 100.0.0.6 remote-as 301
R24(config-router)#exit

R21(config)#router bgp 301
R21(config-router)#neighbor 100.0.0.5 remote-as 520
R21(config-router)#exit
```

Воспользуемся командой <b>show ip bgp summary</b> чтобы посмотреть краткую информацию о состоянии BGP-соединении на конкретном маршрутизаторе:
</code></pre>
</details>
<details>
<summary>show ip bgp summary</summary>
<pre><code>
R24#show ip bgp summary
BGP router identifier 100.0.0.252, local AS number 520
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
100.0.0.6       4          301       4       6        1    0    0 00:01:59        0
</code></pre>
</details>

Мы видим что у маршрутизатора R24 один BGP-соседа с IP-адресом 100.0.0.6 в автономной системе 301.



















<br>

Полные файлы изменений приведены [здесь](config/)
