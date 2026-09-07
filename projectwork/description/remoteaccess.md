### Remote Access VPN

Remote Access VPN — это технология, которая обеспечивает безопасное подключение удаленного пользователя к централизованной корпоративной сети через публичный интернет. Этот тип VPN еще называют VPN удаленного доступа.
<br><br>
![](../picture/remoteaccess-concept.png)
<br><br>
Работает это следующим образом: на устройстве сотрудника — ноутбуке, смартфоне и т.д. — устанавливается специальное программное обеспечение VPN-клиент. Когда пользователь запускает его и проходит аутентификацию, между его устройством и VPN-шлюзом компании создается зашифрованный туннель. Целевой корпоративный трафик с этого устройства направляется через этот туннель, и для внутренних ресурсов сети — файловых серверов, баз данных — сотрудник выглядит так, будто он физически находится в офисе. 

### Удаленный доступ Remote Access к IOS серверу с помощью AnyConnect клиента

При работе с AnyConnect клиентом используется собственная реализация Cisco EAP метода AnyСonnect-EAP. Все коммуникации EAP терминируются на самом сервере FlexVPN. Это отличается от стандартного EAP (например EAP-MD5 или EAP-GTC), когда FlexVPN сервер пробрасывает запросы EAP на внешний AAA сервер (RADIUS).

При этом возможна как локальная аутентификация/авторизация пользователей, так и с использованием внешнего AAA сервера (LDAP). Локальная аутентификация/авторизация довольно удобна для небольших сетей, так не требуется отдельный выделенный AAA сервер.

FlexVPN сервер обязательно должен иметь сертификат, который он предоставляет удаленному пользователю при подключении для проверки подлинности этого сервера. Пользователь же может быть аутентифицирован сервером по имени и паролю, либо своему сертификату.

По умолчанию, AnyConnect использует протокол SSL вместо IPSec, так что потребуется создать отдельный профиль для работы по IPSec. Это можно сделать с помощью редактора профилей AnyConnect «VPN Profile Editor».

Технология на сетевом оборудовании Cisco конфигурируется следующим образом:

1. Включаем HTTP-сервер на маршрутизаторе:
```
ip http server
```

2. Делаем из маршрутизатора CA сервер:
```
crypto pki server FLEXVPN-AnyConnect-CA
 no database archive
 issuer-name cn="FLEXVPN-AnyConnect-CA"
 grant auto
 lifetime certificate 720
 eku server-auth client-auth
 database url flash:certificate
```

3. Создаем на маршрутизаторе сертификат для предъявления клиенту:
```
ip domain name xxx-yyy.ru

crypto pki trustpoint FLEXVPN-AnyConnectTrustPoint-CA
 enrollment url http://xxx.xxx.xxx.xxx:80
 subject-name cn=FLEXVPN-AnyConnectTrustPoint-CA
 revocation-check crl
```
В качестве URL  указывается IP адрес интерфейса, к которому будет подключаться клиент и получать сертификат. Как правило, это публичный "белый" IP адрес интерфейса маршрутизатора.

4. Аутентифицируем  сертификат, и затем запрашиваем клиентский сертификат у маршрутизатора:
```
crypto pki authenticate FLEXVPN-AnyConnectTrustPoint-CA
crypto pki enroll FLEXVPN-AnyConnectTrustPoint-CA
```

5. Создаем два списка для локальной аутентификации и локальной сетевой авторизации, и конечно же пользователя:
```
aaa new-model
aaa authorization network FLEXVPN-AnyConnectAuthorLoc-AAA local
aaa authentication login FLEXVPN-AnyConnectAuthenLoc-AAA local

username test_user privilege 0 algorithm-type sha256 secret !Password!
```

6. Создаем политику авторизации IKEv2 где указываем пул IP адресов для выдачи удаленным пользователям, DNS-сервер, доменное имя и доступные маршруты:
```
ip access-list standard FLEXVPN-AnyConnectRoute-ACL
 remark ===[ Allow routing in subnets: 10.67.1.0, 10.67.2.0, 10.67.2.144, 10.67.4.0, 10.67.4.128 ]===
 permit 10.67.1.0 0.0.0.127
 permit 10.67.2.0 0.0.0.127
 permit 10.67.2.144 0.0.0.15
 permit 10.67.4.0 0.0.0.127
 permit 10.67.4.128 0.0.0.127
 remark ===[ We prohibit everything that is not parted, above ]===
 deny   any

ip local pool FLEXVPN-AnyConnect-POOL 10.67.8.1 10.67.8.254

crypto ikev2 authorization policy FLEXVPN-AnyConnectAuthorizationPolicy-IKEV2 
 pool FLEXVPN-AnyConnect-POOL
 dns 10.67.1.51
 def-domain xxx-yyy.ru
 route set access-list FLEXVPN-AnyConnectRoute-ACL
```

7. Создаем IKEv2 Proposal и затем ее привязываем к IKEv2 Policy:
```
crypto ikev2 proposal FLEXVPN-GeneralProposal-IKEV2
 encryption aes-cbc-256
 integrity sha256
 group 19

crypto ikev2 policy FLEXVPN-GeneralPolicy-IKEV2
 proposal FLEXVPN-GeneralProposal-IKEV2
```

8. Создаем IKEv2 профиль:
```
crypto ikev2 profile FLEXVPN-AnyConnectProfile-IKEV2
 description ===[ Profile of available authentication methods - AnyConnect ]===
 match identity remote key-id *$AnyConnectClient$*
 authentication local rsa-sig
 authentication remote anyconnect-eap aggregate
 pki trustpoint FLEXVPN-AnyConnectTrustPoint-CA
 aaa authentication anyconnect-eap FLEXVPN-AnyConnectAuthenLoc-AAA
 aaa authorization group anyconnect-eap list FLEXVPN-AnyConnectAuthorLoc-AAA FLEXVPN-AnyConnectAuthorizationPolicy-IKEV2
 aaa authorization user anyconnect-eap cached
 virtual-template 100
```

9. Отключаем поиск сертификатов по URL в HTTP запросах, отключаем HTTP сервер:
```
no crypto ikev2 http-url cert
no ip http server
```

10. Собираем воедино профиль IKEv2 и transform-set:
```
crypto ipsec transform-set FLEXVPN-GeneralTransformSet-IPSEC esp-aes 256 esp-sha256-hmac
 mode tunnel

crypto ipsec profile FLEXVPN-AnyConnectProfile-IPSEC
 description ===[ We connect profiles with each other, AnyConnect: FLEXVPN-AnyConnectProfile-IKEV2 & FLEXVPN-GeneralTransformSet-IPSEC ]===
 set transform-set FLEXVPN-GeneralTransformSet-IPSEC
 set ikev2-profile FLEXVPN-AnyConnectProfile-IKEV2
```

11. Описываем виртуальный шаблон который применяется для клонирования конфигурации в Virtual-Access при установлении соединения:
```
interface Loopback100
 description ===[ Tunnel termination - AnyConnect ]===
 ip address 172.16.1.253 255.255.255.255
 zone-member security LAN

interface Virtual-Template100 type tunnel
 description ===[ Dynamic Virtual Tunnel Interface Pattern - AnyConnect ]===
 ip unnumbered Loopback100
 ip mtu 1400
 zone-member security LAN
 ip tcp adjust-mss 1360
 tunnel mode ipsec ipv4
 tunnel path-mtu-discovery
 tunnel protection ipsec profile FLEXVPN-AnyConnectProfile-IPSEC
```
