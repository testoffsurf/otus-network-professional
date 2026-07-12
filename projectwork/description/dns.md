### DNS (Domain Name System)

Как известно в Cisco-маршрутизаторах заложена возможность работать как DNS-сервер - то есть переводить понятные человеку имена (например, mail.ru) в числовые IP-адреса, нужные для маршрутизации.

Чуть ниже я немного расскажу о концепции организации DNS-сервера и на примере конфигурационных команд проиллюстрирую как маршрутизатор Cisco превратить в DNS-сервер.
<br><br>
![](../picture/dns-concept.png)
<br><br>


















```
ip dns server
ip domain-lookup
ip name-server 176.213.132.3 5.3.3.3
```

```
ip dns view-list DNS-DnsAccessRule-VLT
 view default 1
  restrict source access-group DNS-DnsAccessRule-ACL
ip dns server view-group DNS-DnsAccessRule-VLT
```

```
ip access-list extended DNS-DnsAccessRule-ACL
 remark ===[ We allow access to the DNS server to users from the subnet: 10.77.6.0, 10.77.7.0 ]===
 permit ip 10.77.6.0 0.0.0.255 any
 permit ip 10.77.7.0 0.0.0.255 any
 remark ===[ We prohibit everything that is not parted, above ]===
 deny   ip any any
```
