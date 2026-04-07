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
R22(config-router)#neighbor 100.0.0.1 remote-as 520
R22(config-router)#neighbor 100.77.0.1 remote-as 301
R22(config-router)#neighbor 100.78.0.2 remote-as 1001
R22(config-router)#exit

R21(config)#router bgp 301
R21(config-router)#bgp log-neighbor-changes
R21(config-router)#neighbor 100.0.0.5 remote-as 520
R21(config-router)#neighbor 100.77.0.2 remote-as 101
R21(config-router)#neighbor 100.77.0.6 remote-as 1001
R21(config-router)#exit
```

Командой <b>show ip bgp neighbors</b> посмотрим информацию о BGP-соседях:
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
  Message statistics:
    InQ depth is 0
    OutQ depth is 0
  
                         Sent       Rcvd
    Opens:                  1          1
    Notifications:          0          0
    Updates:                2          8
    Keepalives:            94         92
    Route Refresh:          0          0
    Total:                 97        101
  Default minimum time between advertisement runs is 30 seconds

 For address family: IPv4 Unicast
  Session: 100.78.0.1
  BGP table version 8, neighbor version 8/0
  Output queue size : 0
  Index 1, Advertise bit 0
  1 update-group member
  Slow-peer detection is disabled
  Slow-peer split-update-group dynamic is disabled
  Interface associated: Ethernet0/2
                                 Sent       Rcvd
  Prefix activity:               ----       ----
    Prefixes Current:               1          7 (Consumes 560 bytes)
    Prefixes Total:                 1          7
    Implicit Withdraw:              0          0
    Explicit Withdraw:              0          0
    Used as bestpath:             n/a          6
    Used as multipath:            n/a          0

                                   Outbound    Inbound
  Local Policy Denied Prefixes:    --------    -------
    AS_PATH loop:                       n/a          1
    Bestpath from this peer:              6        n/a
    Total:                                6          1
  Number of NLRIs in the update sent: max 1, min 0
  Last detected as dynamic slow peer: never
  Dynamic slow peer recovered: never
  Refresh Epoch: 1
  Last Sent Refresh Start-of-rib: never
  Last Sent Refresh End-of-rib: never
  Last Received Refresh Start-of-rib: never
  Last Received Refresh End-of-rib: never
                                       Sent       Rcvd
        Refresh activity:              ----       ----
          Refresh Start-of-RIB          0          0
          Refresh End-of-RIB            0          0

  Address tracking is enabled, the RIB does have a route to 100.78.0.1
  Connections established 1; dropped 0
  Last reset never
  Transport(tcp) path-mtu-discovery is enabled
  Graceful-Restart is disabled
Connection state is ESTAB, I/O status: 1, unread input bytes: 0
Connection is ECN Disabled, Mininum incoming TTL 0, Outgoing TTL 1
Local host: 100.78.0.2, Local port: 179
Foreign host: 100.78.0.1, Foreign port: 37884
Connection tableid (VRF): 0
Maximum output segment queue size: 50

Enqueued packets for retransmit: 0, input: 0  mis-ordered: 0 (0 bytes)

Event Timers (current time is 0x4EDDA1):
Timer          Starts    Wakeups            Next
Retrans            97          0             0x0
TimeWait            0          0             0x0
AckHold           100         97             0x0
SendWnd             0          0             0x0
KeepAlive           0          0             0x0
GiveUp              0          0             0x0
PmtuAger            0          0             0x0
DeadWait            0          0             0x0
Linger              0          0             0x0
ProcessQ            0          0             0x0

iss:  834199042  snduna:  834200964  sndnxt:  834200964
irs:  636126582  rcvnxt:  636128798

sndwnd:  15928  scale:      0  maxrcvwnd:  16384
rcvwnd:  15643  scale:      0  delrcvwnd:    741

SRTT: 1000 ms, RTTO: 1003 ms, RTV: 3 ms, KRTT: 0 ms
minRTT: 0 ms, maxRTT: 1000 ms, ACK hold: 200 ms
uptime: 5118251 ms, Sent idletime: 16269 ms, Receive idletime: 16476 ms
Status Flags: passive open, gen tcbs
Option Flags: nagle, path mtu capable
IP Precedence value : 6

Datagrams (max data segment is 1460 bytes):
Rcvd: 198 (out of order: 0), with data: 101, total data bytes: 2215
Sent: 198 (retransmit: 0, fastretransmit: 0, partialack: 0, Second Congestion: 0), with data: 97, total data bytes: 1921

 Packets received in fast path: 0, fast processed: 0, slow path: 0
 fast lock acquisition failures: 0, slow path: 0
TCP Semaphore      0xC50531A4  FREE
</code></pre>
</details>

























<br>

Полные файлы изменений приведены [здесь](config/)
