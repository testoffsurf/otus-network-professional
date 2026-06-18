# MPLS. Основные концепции и сервисы

## Цель:
Настроить BGP free core в офисах Москвы и Санкт-Петербурга <br>

## Задание:
  1. Настроите BGP free core в офисе Москвы
  2. Настроите BGP free core в офисе Санкт-Петербурга

### Топология
<center><img src="bgpfreecore.png" align="middle"></center>


### Настроите BGP free core в офисе Москвы
Чтобы запустить BGP free Core в офисе Москва нам необходимо включить MPLS и LDP на соответствующих интерфейсах в сторону нашей сети на всех устройствах в автономной системе. Соответственно, на маршрутизаторах R14 и R15 активируем MPLS на интерфейсах e0/0, e0/1, e0/3. А на маршрутизаторах R12 и R13 активируем MPLS на интерфейсах e0/2, e0/3. 

```
Rxx(config)#mpls ip
Rxx(config)#int range e0/0-1, e0/3
Rxx(config-if-range)#mpls ip
Rxx(config-if-range)#mpls label protocol ldp
Rxx(config-if-range)#exit
```

Воспользуемся командами <b>show mpls interfaces</b>, <b>show mpls ldp neighbor</b>, <b>show mpls forwarding-table</b> и посмотрим что у нас получилось:

</code></pre>
</details>
<details>
<summary>show mpls interfaces</summary>
<pre><code>
R13#show mpls interfaces
Interface              IP            Tunnel   BGP Static Operational
Ethernet0/2            Yes (ldp)     No       No  No     Yes
Ethernet0/3            Yes (ldp)     No       No  No     Yes
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>show mpls ldp neighbor</summary>
<pre><code>
R13#show mpls ldp neighbor
    Peer LDP Ident: 10.77.0.254:0; Local LDP Ident 10.77.0.250:0
        TCP connection: 10.77.0.254.13050 - 10.77.0.250.646
        State: Oper; Msgs sent/rcvd: 37/38; Downstream
        Up time: 00:16:18
        LDP discovery sources:
          Ethernet0/3, Src IP addr: 10.77.0.9
        Addresses bound to peer LDP Ident:
          10.77.0.5       10.77.0.254     10.77.0.9       100.78.0.2
          10.77.0.1
    Peer LDP Ident: 10.77.0.253:0; Local LDP Ident 10.77.0.250:0
        TCP connection: 10.77.0.253.13232 - 10.77.0.250.646
        State: Oper; Msgs sent/rcvd: 38/40; Downstream
        Up time: 00:16:14
        LDP discovery sources:
          Ethernet0/2, Src IP addr: 10.77.0.17
        Addresses bound to peer LDP Ident:
          10.77.0.17      10.77.0.253     10.77.0.13      100.77.0.6
          10.77.0.21      172.16.1.254    172.16.0.1
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>show mpls forwarding-table</summary>
<pre><code>
R13#show mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop
Label      Label      or Tunnel Id     Switched      interface
20         Pop Label  10.77.0.254/32   0             Et0/3      10.77.0.9
21         Pop Label  10.77.0.253/32   3207          Et0/2      10.77.0.17
22         Pop Label  172.16.1.0/24    0             Et0/2      10.77.0.17
23         Pop Label  10.77.0.4/30     0             Et0/3      10.77.0.9
24         Pop Label  10.77.0.12/30    0             Et0/2      10.77.0.17
25         24         10.77.0.251/32   0             Et0/3      10.77.0.9
           21         10.77.0.251/32   0             Et0/2      10.77.0.17
26         28         10.77.0.28/30    0             Et0/3      10.77.0.9
           22         10.77.0.28/30    0             Et0/2      10.77.0.17
27         29         10.77.0.24/30    0             Et0/3      10.77.0.9
           23         10.77.0.24/30    0             Et0/2      10.77.0.17
28         Pop Label  10.77.0.20/30    0             Et0/2      10.77.0.17
29         Pop Label  10.77.0.0/30     0             Et0/3      10.77.0.9
</code></pre>
</details>

### Настроите BGP free core в офисе Санкт-Петербурга
Чтобы запустить BGP free Core в офисе Санкт-Петербурга нам необходимо включить MPLS и LDP на соответствующих интерфейсах в сторону нашей сети на всех устройствах в автономной системе. Соответственно, на маршрутизаторе R18 активируем MPLS на интерфейсах e0/0, e0/1. А на маршрутизаторах R16 и R17 активируем MPLS на интерфейсе e0/1.

```
R18(config)#mpls ip
R18(config)#int range e0/0-1
R18(config-if-range)#mpls ip
R18(config-if-range)#mpls label protocol ldp
R18(config-if-range)#exit
```

```
R16(config)#mpls ip
R16(config)#int e0/1
R16(config-if-range)#mpls ip
R16(config-if-range)#mpls label protocol ldp
R16(config-if-range)#exit
```

```
R17(config)#mpls ip
R17(config)#int e0/1
R17(config-if-range)#mpls ip
R17(config-if-range)#mpls label protocol ldp
R17(config-if-range)#exit
```

Воспользуемся командами <b>show mpls interfaces</b>, <b>show mpls ldp neighbor</b>, <b>show mpls forwarding-table</b> и посмотрим что у нас получилось:

</code></pre>
</details>
<details>
<summary>show mpls interfaces</summary>
<pre><code>
R18#show mpls interfaces
Interface              IP            Tunnel   BGP Static Operational
Ethernet0/0            Yes (ldp)     No       No  No     Yes
Ethernet0/1            Yes (ldp)     No       No  No     Yes
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>show mpls ldp neighbor</summary>
<pre><code>
R18#show mpls ldp neighbor
    Peer LDP Ident: 10.78.0.253:0; Local LDP Ident 10.78.0.254:0
        TCP connection: 10.78.0.253.646 - 10.78.0.254.62138
        State: Oper; Msgs sent/rcvd: 40/37; Downstream
        Up time: 00:21:53
        LDP discovery sources:
          Ethernet0/1, Src IP addr: 10.78.0.26
        Addresses bound to peer LDP Ident:
          10.78.0.21      10.78.0.26      10.78.0.253     10.78.0.17
    Peer LDP Ident: 10.78.0.252:0; Local LDP Ident 10.78.0.254:0
        TCP connection: 10.78.0.252.646 - 10.78.0.254.38655
        State: Oper; Msgs sent/rcvd: 39/38; Downstream
        Up time: 00:21:51
        LDP discovery sources:
          Ethernet0/0, Src IP addr: 10.78.0.2
        Addresses bound to peer LDP Ident:
          10.78.0.9       10.78.0.2       10.78.0.252     10.78.0.13
          10.78.0.5
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>show mpls forwarding-table</summary>
<pre><code>
R18#show mpls forwarding-table
Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop
Label      Label      or Tunnel Id     Switched      interface
20         Pop Label  10.78.0.253/32   0             Et0/1      10.78.0.26
21         Pop Label  10.78.0.16/28    0             Et0/1      10.78.0.26
22         Pop Label  10.78.0.252/32   0             Et0/0      10.78.0.2
23         Pop Label  10.78.0.0/28     0             Et0/0      10.78.0.2
</code></pre>
</details>

<br>

Полные файлы изменений приведены [здесь](config/)
