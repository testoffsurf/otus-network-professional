# Сертификаты. Часть 2: практика. TLS/SSL

## Цель:
Настроить GRE поверх IPSec между офисами Москва и Санкт-Петербург <br>
Настроить DMVPN поверх IPSec между офисами Москва и Чокурдах, Лабытнанги <br>

## Задание:
  1. Настроите GRE поверх IPSec между офисами Москва и Санкт-Петербург
  2. Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги <br>
  Дополнительно: Для IPSec использовать CA и сертификаты

### Топология
<center><img src="crt.png" align="middle"></center>


### Сервер сертификации и сертификаты
В качестве сервера сертификации выберем маршрутизатор R14 в Московсом офисе. Воспользуемся следующими конфигурационными командами чтобы активировать сервер сертификации на маршрутизаторе R14:

```
R14(config)#ip domain name laba.ru
R14(config)#ip http server
R14(config)#crypto key generate rsa general-keys label R14 exportable modulus 2048

R14(config)#crypto pki server R14
R14(cs-server)#database level complete
R14(cs-server)#issuer-name CN=R14, O=laba, C=ru
R14(cs-server)#no shutdown
R14(cs-server)#exit
```

Воспользуемся командой <b>show crypto pki server</b> чтобы убедиться что у нас на маршрутизаторе запущен и работает центр сертификации, а также получить информацию о его состоянии и конфигурации.

</code></pre>
</details>
<details>
<summary>show crypto pki server</summary>
<pre><code>
R14#show crypto pki server
Certificate Server R14:
    Status: enabled
    State: enabled
    Server's configuration is locked  (enter "shut" to unlock it)
    Issuer name: CN=R14, O=laba, C=ru
    <b>CA cert fingerprint: 3F0B88FA 32299C7F C46A3DA7 2BE08A8E</b>
    Granting mode is: manual
    Last certificate issued serial number (hex): 1
    CA certificate expiration timer: 06:42:59 UTC Jun 29 2029
    CRL NextUpdate timer: 12:42:59 UTC Jun 30 2026
    Current primary storage dir: nvram:
    Database Level: Complete - all issued certs written as <serialnum>.cer
</code></pre>
</details>

И так мы видим, что центр сертификации на R14 запушен и работает.

Теперь нам необходимо получить сертификаты для устройств которые участвуют в обмене информации через IPSec + GRE. Для этого мы на выбранном устройстве ввидем следующие конфигурационные команды:

```
R15(config)#ip host R14 1.1.1.14
R15(config)#ip host R14.laba.ru 1.1.1.14

R15(config)#crypto pki trustpoint R14
R15(ca-trustpoint)#enrollment url http://R14.laba.ru:80
R15(ca-trustpoint)#serial-number
R15(ca-trustpoint)#subject-name CN=R15, O=laba, O=ru
R15(ca-trustpoint)#revocation-check crl
R15(ca-trustpoint)#exit

R15(config)#crypto pki authenticate R14
Certificate has the following attributes:
       <b>Fingerprint MD5: 3F0B88FA 32299C7F C46A3DA7 2BE08A8E</b>
      Fingerprint SHA1: E21BFD7B 2FBBFB97 A4930A98 E9E65994 5AFD7E6B

% Do you accept this certificate? [yes/no]: y
Trustpoint CA certificate accepted.

R15(config)#crypto pki enroll R14

<b>.....</b>

R15(config)#
000031: *Jun 30 2026 09:20:10.643 UTC: CRYPTO_PKI:  Certificate Request Fingerprint MD5: C23F2612 816D4778 F1621211 5B234310
000032: *Jun 30 2026 09:20:10.643 UTC: CRYPTO_PKI:  Certificate Request Fingerprint SHA1: 44EFEC65 79A6DD30 3FF2A336 7878AC40 9A94E48A
R15(config)#
```

Проверяем появился ли в центре сертификации запрос от маршрутизатора R15 на подтверждение сертификата:

</code></pre>
</details>
<details>
<summary>R14</summary>
<pre><code>
```
R14#show crypto pki server R14 requests
Enrollment Request Database:
<br>
Subordinate CA certificate requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------
<br>
RA certificate requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------
<br>
Router certificates requests:
ReqID  State      Fingerprint                      SubjectName
--------------------------------------------------------------
1      pending    C23F2612816D4778F16212115B234310 serialNumber=67109104+hostname=R15,cn=R15,o=laba,o=ru
<br>
R14#crypto pki server R14 grant 1
<br>
R14#show crypto pki server R14 certificates
Serial Issued date              Expire date               Subject Name
1       06:42:59 UTC Jun 30 2026 06:42:59 UTC Jun 29 2029  cn=R14 o=laba c=ru
2       10:13:01 UTC Jun 30 2026 10:13:01 UTC Jun 30 2027  serialNumber=67109104+hostname=R15 cn=R15 o=laba o=ru
```
</code></pre>
</details>

### Настроите GRE поверх IPSec между офисами Москва и Санкт-Петербург
Для того чтобы настроить GRE-туннель поверх IPSec между офисами Москва и Санкт-Петербург, воспользуемся следующими конфигурационными командами:

</code></pre>
</details>
<details>
<summary>R15</summary>
<pre><code>
```
R15(config)#crypto ikev2 proposal LABA-PROPOSAL
R15(config-ikev2-proposal)#encryption aes-cbc-256
R15(config-ikev2-proposal)#integrity sha256
R15(config-ikev2-proposal)#group 19
R15(config-ikev2-proposal)#exit
<br>
R15(config)#crypto ikev2 policy LABA-POLICY
R15(config-ikev2-policy)#proposal LABA-PROPOSAL
R15(config-ikev2-policy)#exit
<br>
R15(config)#crypto ikev2 profile LABA-PROFILE
R15(config-ikev2-profile)#match identity remote address 0.0.0.0
R15(config-ikev2-profile)#authentication remote rsa-sig
R15(config-ikev2-profile)#authentication local rsa-sig
R15(config-ikev2-profile)#pki trustpoint R14
R15(config-ikev2-profile)#exit
<br>
R15(config)#crypto ipsec transform-set LABA-TS esp-aes esp-sha256-hmac
R15(cfg-crypto-trans)#mode transport
R15(cfg-crypto-trans)#exit
<br>
R15(config)#crypto ipsec profile LABA-IPSEC-PROFILE
R15(ipsec-profile)#set transform-set LABA-TS
R15(ipsec-profile)#set ikev2-profile LABA-PROFILE
R15(ipsec-profile)#exit
<br>
R15(config)#interface tunnel 0
R15(config-if)#tunnel protection ipsec profile LABA-IPSEC-PROFILE
R15(config-if)#exit
```
</code></pre>
</details>
<br>
</code></pre>
</details>
<details>
<summary>R18</summary>
<pre><code>
```
R18(config)#crypto ikev2 proposal LABA-PROPOSAL
R18(config-ikev2-proposal)#encryption aes-cbc-256
R18(config-ikev2-proposal)#integrity sha256
R18(config-ikev2-proposal)#group 19
R18(config-ikev2-proposal)#exit
<br>
R18(config)#crypto ikev2 policy LABA-POLICY
R18(config-ikev2-policy)#proposal LABA-PROPOSAL
R18(config-ikev2-policy)#exit
<br>
R18(config)#crypto ikev2 profile LABA-PROFILE
R18(config-ikev2-profile)#match identity remote address 0.0.0.0
R18(config-ikev2-profile)#authentication remote rsa-sig
R18(config-ikev2-profile)#authentication local rsa-sig
R18(config-ikev2-profile)#pki trustpoint R14
R18(config-ikev2-profile)#exit
<br>
R18(config)#crypto ipsec transform-set LABA-TS esp-aes esp-sha256-hmac
R18(cfg-crypto-trans)#mode transport
R18(cfg-crypto-trans)#exit
<br>
R18(config)#crypto ipsec profile LABA-IPSEC-PROFILE
R18(ipsec-profile)#set transform-set LABA-TS
R18(ipsec-profile)#set ikev2-profile LABA-PROFILE
R18(ipsec-profile)#exit
<br>
R18(config)#interface tunnel 0
R18(config-if)#tunnel protection ipsec profile LABA-IPSEC-PROFILE
R18(config-if)#exit
```
</code></pre>
</details>

Воспользуемся командой <b>show crypto ipsec sa</b> чтобы посмотреть информацию об IPSec соединении.
<br>
</code></pre>
</details>
<details>
<summary>show crypto ipsec sa</summary>
<pre><code>
```
show crypto ipsec sa  
```
</code></pre>
</details>




### Настроите DMVPN поверх IPSec между Москва и Чокурдах, Лабытнанги



<br>

Полные файлы изменений приведены [здесь](config/)
