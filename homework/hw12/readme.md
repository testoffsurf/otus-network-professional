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
  
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>show mpls ldp neighbor</summary>
<pre><code>
  
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>show mpls forwarding-table</summary>
<pre><code>
  
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
  
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>show mpls ldp neighbor</summary>
<pre><code>
  
</code></pre>
</details>

</code></pre>
</details>
<details>
<summary>show mpls forwarding-table</summary>
<pre><code>
  
</code></pre>
</details>

<br>

Полные файлы изменений приведены [здесь](config/)
