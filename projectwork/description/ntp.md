### NTP (Network Time Protocol)

Как известно в Cisco-маршрутизаторах заложена возможность работать как DNS-сервер - то есть переводить понятные человеку имена (например, mail.ru) в числовые IP-адреса, нужные для маршрутизации.

Чуть ниже я немного расскажу о концепции организации DNS-сервера и на примере конфигурационных команд проиллюстрирую как маршрутизатор Cisco превратить в DNS-сервер.
<br><br>
![](../picture/ntp-concept.png)
<br><br>
Концепция организации DNS-сервера в небольшом офисе достаточно проста, ввиду отсутствия в малом офисе серверов Linux или Windows c поднятым DNS-сервером этот функционал мы поручим исполнять маршрутизатору. Когда клиент в такой сети (рабочая станция, другое IP-оборудование) захочет узнать IP-адрес по имени, он отправит соответствующий запрос на маршрутизатор. Увидев такой запрос маршрутизатор будет его обрабатывать следующим образом: если запрошенное имя есть в локальной таблице маршрутизатора, то клиент практически сразу получит его IP-адрес. Если запрошенного имени в таблице нет, маршрутизатор перешлет запрос дальше - на другой DNS-сервер (например, на DNS-сервер провайдера), чтобы тот нашел ответ. Такой принцип организации DNS-сервера используется в удаленных филиалах предприятия, в центральном же офисе предприятия все иначе.







```
ntp logging
ntp source Port-channel1.998
ntp access-group peer NTP-NtpServerSourcePeers-ACL kod
ntp access-group serve NTP-NtpClientsDestinationPeers-ACL kod
ntp master 3
ntp server 194.190.168.1 prefer source GigabitEthernet0/0/1
ntp server 88.147.254.235 source GigabitEthernet0/0/1
ntp server 88.147.254.227 source GigabitEthernet0/0/1
ntp server 88.147.254.229 source GigabitEthernet0/0/1
```

```
ip access-list extended NTP-NtpClientsDestinationPeers-ACL
 remark ===[ Allow access to the internal NTP server from the subnet: 10.77.5.0 - 10.77.5.255 ]=== 
 permit udp 10.77.5.0 0.0.0.255 host 10.77.5.1 eq ntp
 remark ===[ Allow access to the internal NTP server from the subnet: 10.77.6.0 - 10.77.6.255 ]===
 permit udp 10.77.6.0 0.0.0.255 host 10.77.5.1 eq ntp
 remark ===[ Allow access to the internal NTP server from the subnet: 10.77.7.0 - 10.77.7.255 ]===
 permit udp 10.77.7.0 0.0.0.255 host 10.77.5.1 eq ntp
 remark ===[ We prohibit everything that is not parted, above ]===
 deny   ip any any
```

```
ip access-list extended NTP-NtpServerSourcePeers-ACL
 remark ===[ Allow access to external NTP server - ntp.msk-ix.ru ]===
 permit udp host 194.190.168.1 any
 remark ===[ Allow access to external NTP server - ntp0.ntp-servers.net ]===
 permit udp host 88.147.254.227 any
 remark ===[ Allow access to external NTP server - ntp3.ntp-servers.net ]===
 permit udp host 88.147.254.229 any
 remark ===[ Allow access to external NTP server - ntp6.ntp-servers.net ]===
 permit udp host 88.147.254.235 any
 remark ===[ We prohibit everything that is not parted, above ]===
 deny   ip any any
```







