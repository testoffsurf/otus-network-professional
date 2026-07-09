### SNMP (Simple Network Management Protocol)

SNMP-протокол позволяет проводить мониторинг, контролировать производительность сети и изменять конфигурацию подключенных устройств. SNMP используют в сетях любого размера: чем крупнее сеть, тем лучше раскрываются преимущества протокола. Он позволяет просматривать, контролировать и управлять узлами через единый интерфейс с функциями пакетных команд и автоматического оповещения.



### Схема иерархии SNMP
![](../picture/snmp-concept.png)






<br>
<br>
<br>




```
snmp-server community CiscoPublic RO SNMP-SnmpAccessHost-ACL
snmp-server community CiscoPrivate RW SNMP-SnmpAccessHost-ACL
snmp-server location Moscow
snmp-server contact SysEng <1027@xxx-yyy.ru>
snmp-server host 10.67.1.34 version 2c CiscoPrivate
snmp-server host 10.67.1.34 version 2c CiscoPublic
snmp mib flash cache
```

Как и любой другой сервис 


```
ip access-list standard SNMP-SnmpAccessHost-ACL
 remark ===[ Allowing access to the SNMP protocol from an IP address: 10.67.1.34 (APP-XXXXXX04) ]===
 permit 10.67.1.34
 remark ===[ We prohibit everything that is not parted, above ]===
 deny   any
```


