# BGP. Управление анонсами

## Цель:
Настроить фильтрацию для офисе Москва <br>
Настроить фильтрацию для офисе С.-Петербург <br>

## Задание:
  1. Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика (As-path).
  2. Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика (Prefix-list).
  3. Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.
  4. Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса С.-Петербург.

### Топология
<center><img src="bgp_topologi.png" align="middle"></center>

<br>

### Настроить фильтрацию в офисе Москва так, чтобы не появилось транзитного трафика (As-path)
Посмотрим какие префиксы отдаются в сторону интернет провайдеров Киторн и Ламас из Московского офиса:
<details>
<summary>R22</summary>
<pre><code>
R22#sh ip bgp neighbors 100.78.0.2 received-routes
BGP table version is 40, local router ID is 100.78.0.254
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found
       Network          Next Hop            Metric LocPrf Weight Path
 *>  10.77.0.0/30     100.78.0.2               0             0 1001 ?
 *>  10.77.0.4/30     100.78.0.2               0             0 1001 ?
 *>  10.77.0.8/30     100.78.0.2               0             0 1001 ?
 *>  10.77.0.12/30    100.78.0.2              20             0 1001 ?
 *>  10.77.0.16/30    100.78.0.2              20             0 1001 ?
 *>  10.77.0.20/30    100.78.0.2              30             0 1001 ?
 *>  10.77.0.24/30    100.78.0.2              20             0 1001 ?
 *>  10.77.0.28/30    100.78.0.2              20             0 1001 ?
 *>  10.77.0.32/30    100.78.0.2              20             0 1001 ?
 *>  10.77.0.36/30    100.78.0.2              20             0 1001 ?
 *>  10.77.0.249/32   100.78.0.2              31             0 1001 ?
 *>  10.77.0.250/32   100.78.0.2              11             0 1001 ?
 *>  10.77.0.251/32   100.78.0.2              11             0 1001 ?
 *>  10.77.0.252/32   100.78.0.2              11             0 1001 ?
 *>  10.77.0.253/32   100.78.0.2              21             0 1001 ?
 *>  10.77.0.254/32   100.78.0.2               0             0 1001 ?
 &#42;   10.78.0.0/30     100.78.0.2                             0 1001 301 520 2042 ?
 &#42;   10.78.0.0/28     100.78.0.2                             0 1001 301 520 2042 ?
 &#42;   10.78.0.16/28    100.78.0.2                             0 1001 301 520 2042 ?
 &#42;   10.78.0.24/30    100.78.0.2                             0 1001 301 520 2042 ?
 &#42;   10.78.0.251/32   100.78.0.2                             0 1001 301 520 2042 ?
 &#42;   10.78.0.252/32   100.78.0.2                             0 1001 301 520 2042 ?
 &#42;   10.78.0.253/32   100.78.0.2                             0 1001 301 520 2042 ?
 &#42;   10.78.0.254/32   100.78.0.2                             0 1001 301 520 2042 ?
 &#42;   100.0.0.0/30     100.78.0.2                             0 1001 301 520 i
 &#42;   100.0.0.4/30     100.78.0.2                             0 1001 301 i
 &#42;   100.0.0.8/30     100.78.0.2                             0 1001 301 520 i
 &#42;   100.0.0.20/30    100.78.0.2                             0 1001 301 520 i
 &#42;   100.77.0.0/30    100.78.0.2                             0 1001 301 i
 &#42;   100.77.0.4/30    100.78.0.2                             0 1001 i
 &#42;   100.78.0.0/30    100.78.0.2               0             0 1001 i
Total number of prefixes 31
</code></pre>
</details>

<details>
<summary>R21</summary>
<pre><code>
R21#sh ip bgp neighbors 100.77.0.6 received-routes
BGP table version is 32, local router ID is 100.77.0.254
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found
     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.77.0.0/30     100.77.0.6              30             0 1001 ?
 *>  10.77.0.4/30     100.77.0.6              20             0 1001 ?
 *>  10.77.0.8/30     100.77.0.6              20             0 1001 ?
 *>  10.77.0.12/30    100.77.0.6               0             0 1001 ?
 *>  10.77.0.16/30    100.77.0.6               0             0 1001 ?
 *>  10.77.0.20/30    100.77.0.6               0             0 1001 ?
 *>  10.77.0.24/30    100.77.0.6              20             0 1001 ?
 *>  10.77.0.28/30    100.77.0.6              20             0 1001 ?
 *>  10.77.0.32/30    100.77.0.6              20             0 1001 ?
 *>  10.77.0.36/30    100.77.0.6              20             0 1001 ?
 *>  10.77.0.249/32   100.77.0.6              11             0 1001 ?
 *>  10.77.0.250/32   100.77.0.6              11             0 1001 ?
 *>  10.77.0.251/32   100.77.0.6              11             0 1001 ?
 *>  10.77.0.252/32   100.77.0.6              31             0 1001 ?
 *>  10.77.0.253/32   100.77.0.6               0             0 1001 ?
 *>  10.77.0.254/32   100.77.0.6              21             0 1001 ?
 &#42;   100.77.0.4/30    100.77.0.6               0             0 1001 i
 *>  100.78.0.0/30    100.77.0.6                             0 1001 i
Total number of prefixes 18
</code></pre>
</details>

И так в сторону интернет провайдера Киторн отдается 31 префикс, а в сторону интернет провайдера Ламас 18 префиксов.

<br>

Для того чтобы в офисе Москва не появился транзитный трафик нам необходимо написать регулярное выражение для фильтрации маршрутов:

```
R14(config)#ip as-path access-list 1 permit ^$
R14(config)#ip as-path access-list 1 deny .*
R14(config)#router bgp 1001
R14(config-router)#neighbor 100.78.0.1 filter-list 1 out
R14(config-router)#exit

R15(config)#ip as-path access-list 1 permit ^$
R15(config)#ip as-path access-list 1 deny .*
R15(config)#router bgp 1001
R15(config-router)#neighbor 100.77.0.5 filter-list 1 out
R15(config-router)#exit
```

Посмотрим что изменилось после применения регулярного выражения:
<details>
<summary>R22</summary>
<pre><code>
R22#sh ip bgp neighbors 100.78.0.2 received-routes
BGP table version is 1, local router ID is 100.78.0.254
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found
     Network          Next Hop            Metric LocPrf Weight Path
 &#42;   10.77.0.0/30     100.78.0.2               0             0 1001 ?
 &#42;   10.77.0.4/30     100.78.0.2               0             0 1001 ?
 &#42;   10.77.0.8/30     100.78.0.2               0             0 1001 ?
 &#42;   10.77.0.12/30    100.78.0.2              20             0 1001 ?
 &#42;   10.77.0.16/30    100.78.0.2              20             0 1001 ?
 &#42;   10.77.0.20/30    100.78.0.2              30             0 1001 ?
 &#42;   10.77.0.24/30    100.78.0.2              20             0 1001 ?
 &#42;   10.77.0.28/30    100.78.0.2              20             0 1001 ?
 &#42;   10.77.0.32/30    100.78.0.2              20             0 1001 ?
 &#42;   10.77.0.36/30    100.78.0.2              20             0 1001 ?
 &#42;   10.77.0.249/32   100.78.0.2              31             0 1001 ?
 &#42;   10.77.0.250/32   100.78.0.2              11             0 1001 ?
 &#42;   10.77.0.251/32   100.78.0.2              11             0 1001 ?
 &#42;   10.77.0.252/32   100.78.0.2              11             0 1001 ?
 &#42;   10.77.0.253/32   100.78.0.2              21             0 1001 ?
 &#42;   10.77.0.254/32   100.78.0.2               0             0 1001 ?
 &#42;   100.77.0.4/30    100.78.0.2                             0 1001 i
 &#42;   100.78.0.0/30    100.78.0.2               0             0 1001 i
Total number of prefixes 18
</code></pre>
</details>

<details>
<summary>R21</summary>
<pre><code>
R21#sh ip bgp neighbors 100.77.0.6 received-routes
BGP table version is 32, local router ID is 100.77.0.254
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found
     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.77.0.0/30     100.77.0.6              30             0 1001 ?
 *>  10.77.0.4/30     100.77.0.6              20             0 1001 ?
 *>  10.77.0.8/30     100.77.0.6              20             0 1001 ?
 *>  10.77.0.12/30    100.77.0.6               0             0 1001 ?
 *>  10.77.0.16/30    100.77.0.6               0             0 1001 ?
 *>  10.77.0.20/30    100.77.0.6               0             0 1001 ?
 *>  10.77.0.24/30    100.77.0.6              20             0 1001 ?
 *>  10.77.0.28/30    100.77.0.6              20             0 1001 ?
 *>  10.77.0.32/30    100.77.0.6              20             0 1001 ?
 *>  10.77.0.36/30    100.77.0.6              20             0 1001 ?
 *>  10.77.0.249/32   100.77.0.6              11             0 1001 ?
 *>  10.77.0.250/32   100.77.0.6              11             0 1001 ?
 *>  10.77.0.251/32   100.77.0.6              11             0 1001 ?
 *>  10.77.0.252/32   100.77.0.6              31             0 1001 ?
 *>  10.77.0.253/32   100.77.0.6               0             0 1001 ?
 *>  10.77.0.254/32   100.77.0.6              21             0 1001 ?
 &#42;   100.77.0.4/30    100.77.0.6               0             0 1001 i
 *>  100.78.0.0/30    100.77.0.6                             0 1001 i
Total number of prefixes 18
</code></pre>
</details>

Как видим, число получаемых маршрутизатором R22 маршрутов от маршрутизатора R14 снизилось с 31 до 18.

### Настроить фильтрацию в офисе С.-Петербург так, чтобы не появилось транзитного трафика (Prefix-list)
Посмотрим какие префиксы отдаются в сторону интернет провайдера Триада из Санкт-Петербурского офиса:
<details>
<summary>R24</summary>
<pre><code>
R24#sh ip bgp neighbors 100.0.0.10 received-routes
BGP table version is 62, local router ID is 100.0.0.252
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found
     Network          Next Hop            Metric LocPrf Weight Path
 *>  10.78.0.0/30     100.0.0.10               0             0 2042 ?
 *>  10.78.0.0/28     100.0.0.10         1536000             0 2042 ?
 *>  10.78.0.8/30     100.0.0.10         2048000             0 2042 ?
 *>  10.78.0.12/30    100.0.0.10         2048000             0 2042 ?
 *>  10.78.0.16/30    100.0.0.10         2048000             0 2042 ?
 *>  10.78.0.16/28    100.0.0.10         1536000             0 2042 ?
 *>  10.78.0.20/30    100.0.0.10         2048000             0 2042 ?
 *>  10.78.0.24/30    100.0.0.10               0             0 2042 ?
 *>  10.78.0.249/32   100.0.0.10         1536640             0 2042 ?
 *>  10.78.0.250/32   100.0.0.10         1536640             0 2042 ?
 *>  10.78.0.251/32   100.0.0.10         1536640             0 2042 ?
 *>  10.78.0.252/32   100.0.0.10         1024640             0 2042 ?
 *>  10.78.0.253/32   100.0.0.10         1024640             0 2042 ?
 *>  10.78.0.254/32   100.0.0.10               0             0 2042 ?
 *>  10.78.1.0/24     100.0.0.10         1541120             0 2042 ?
 &#42;   100.0.0.8/30     100.0.0.10               0             0 2042 i
 r   100.0.0.20/30    100.0.0.10               0             0 2042 i
Total number of prefixes 17
</code></pre>
</details>
И так в сторону интернет провайдера Триада, Санкт-Петербургский офис отдается 17 префиксов.<br><br>

Настроим фильтрацию префиксов через <b>ip prefix-list</b> таким образом чтобы не появился транзитный трафик:
```
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 10 deny 10.78.0.0/30
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 15 deny 10.78.0.0/28
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 20 deny 10.78.0.8/30
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 25 deny 10.78.0.12/30
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 30 deny 10.78.0.16/30
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 35 deny 10.78.0.16/28
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 40 deny 10.78.0.20/30
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 45 deny 10.78.0.24/30
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 50 deny 10.78.0.249/32
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 55 deny 10.78.0.250/32
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 60 deny 10.78.0.251/32
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 65 deny 10.78.0.252/32
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 70 deny 10.78.0.253/32
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 75 deny 10.78.0.254/32
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 80 deny 10.78.1.0/24
R18(config)#ip prefix-list SPBFILTER-TO-TRIADA seq 85 permit 0.0.0.0/0 le 32

R18(config)#router bgp 2042
R18(config-router)#neighbor 100.0.0.9 prefix-list SPBFILTER-TO-TRIADA out
R18(config-router)#neighbor 100.0.0.21 prefix-list SPBFILTER-TO-TRIADA out
R18(config-router)#exit
```

Посмотрим что изменилось после применения префикс-листа:
<details>
<summary>R24</summary>
<pre><code>
R24#sh ip bgp neighbors 100.0.0.10 received-routes
BGP table version is 92, local router ID is 100.0.0.252
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found
     Network          Next Hop            Metric LocPrf Weight Path
 &#42;   100.0.0.8/30     100.0.0.10               0             0 2042 i
 r   100.0.0.20/30    100.0.0.10               0             0 2042 i
Total number of prefixes 2
</code></pre>
</details>

И так в сторону интернет провайдера Траида отдается 2 префикса, вместо 17 ранее.

### Настроить провайдера Киторн так, чтобы в офис Москва отдавался только маршрут по умолчанию.
Посмотрим какие префиксы отдаются со стороны интернет провайдера Киторн в Московский офис:
<details>
<summary>R14</summary>
<pre><code>
R14#sh ip bgp neighbors 100.78.0.1 received-routes
BGP table version is 52, local router ID is 10.77.0.254
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found
     Network          Next Hop            Metric LocPrf Weight Path
 &#42;   100.0.0.0/30     100.78.0.1               0             0 101 i
 &#42;   100.0.0.4/30     100.78.0.1                             0 101 520 i
 &#42;   100.0.0.8/30     100.78.0.1                             0 101 520 i
 &#42;   100.0.0.20/30    100.78.0.1                             0 101 520 i
 &#42;   100.77.0.0/30    100.78.0.1               0             0 101 i
 &#42;   100.78.0.0/30    100.78.0.1               0             0 101 i
Total number of prefixes 6
</code></pre>
</details>

И так со стороны интернет провайдера Киторн в Московский офис отдается 6 префиксов.

<br>

Для того чтобы со стороны интернет провайдера Киторн в Московский офис отдавался только маршрут по умолчанию необходимо сконфигурировать маршрутизатор R22 следующим образом:
```
R22(config)#router bgp 101
R22(config-router)#neighbor 100.78.0.2 default-originate
R22(config-router)#exit
```

Посмотрим что изменилось после применения, вышеописанного конфигурационного кода:
<details>
<summary>R14</summary>
<pre><code>
R14#sh ip bgp neighbors 100.78.0.1 received-routes
BGP table version is 42, local router ID is 10.77.0.254
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found
     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          100.78.0.1                             0 101 i
 &#42;   100.0.0.0/30     100.78.0.1               0             0 101 i
 &#42;   100.0.0.4/30     100.78.0.1                             0 101 520 i
 &#42;   100.0.0.8/30     100.78.0.1                             0 101 520 i
 &#42;   100.0.0.20/30    100.78.0.1                             0 101 520 i
 &#42;   100.77.0.0/30    100.78.0.1               0             0 101 i
 &#42;   100.78.0.0/30    100.78.0.1               0             0 101 i
Total number of prefixes 7
</code></pre>
</details>

Мы видим что в таблице появился маршрут по умолчанию и он считается наилучшим, но также мы замечаем в таблице присутствуют и другие маршруты которые не совсем нам кстати. Возьмем и отфильтруем их, чтобы получить то что нам необходимо:
```
R22(config)#ip prefix-list KITORNFILTER-TO-MOSCOW seq 10 deny 100.0.0.0/30
R22(config)#ip prefix-list KITORNFILTER-TO-MOSCOW seq 15 deny 100.0.0.4/30
R22(config)#ip prefix-list KITORNFILTER-TO-MOSCOW seq 20 deny 100.0.0.8/30
R22(config)#ip prefix-list KITORNFILTER-TO-MOSCOW seq 25 deny 100.0.0.20/30
R22(config)#ip prefix-list KITORNFILTER-TO-MOSCOW seq 30 deny 100.77.0.0/30
R22(config)#ip prefix-list KITORNFILTER-TO-MOSCOW seq 35 deny 100.78.0.0/30
R22(config)#ip prefix-list KITORNFILTER-TO-MOSCOW seq 40 permit 0.0.0.0/0 le 32

R22(config)#router bgp 101
R22(config-router)#neighbor 100.78.0.2 prefix-list KITORNFILTER-TO-MOSCOW out
R22(config-router)#exit
```

Посмотрим что из этого получилось:
<details>
<summary>R14</summary>
<pre><code>
R14#sh ip bgp neighbors 100.78.0.1 received-routes
BGP table version is 44, local router ID is 10.77.0.254
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found
     Network          Next Hop            Metric LocPrf Weight Path
 *>  0.0.0.0          100.78.0.1                             0 101 i
Total number of prefixes 1
</code></pre>
</details>

Теперь то что требовалось.

### Настроить провайдера Ламас так, чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса Санкт-Петербург
Маршрутизатор R21 у интернет провайдера Ламас сконфигурируем следующим образом (чтобы в офис Москва отдавался только маршрут по умолчанию и префикс офиса Санкт-Петербург):
```
R21(config)#router bgp 301
R21(config-router)#neighbor 100.77.0.6 remote-as 1001
R21(config-router)#neighbor 100.77.0.6 default-originate
R21(config-router)#neighbor 100.77.0.6 soft-reconfiguration inbound
R21(config-router)#neighbor 100.77.0.6 route-map LAMAS-TO-MOSCOW out
R21(config-router)#exit

R21(config)#ip as-path access-list 1 permit _2042$
R21(config)#ip prefix-list DEF-ORIGINATE seq 10 permit 0.0.0.0/0

R21(config)#route-map LAMAS-TO-MOSCOW permit 10
R21(config-route-map)#match ip address prefix-list DEF-ORIGINATE
R21(config-route-map)#exit

R21(config)#route-map LAMAS-TO-MOSCOW permit 20
R21(config-route-map)#match as-path 1
R21(config-route-map)#exit
```

<br>

Полные файлы изменений приведены [здесь](config/)
