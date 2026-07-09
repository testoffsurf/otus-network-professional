### SNMP (Simple Network Management Protocol)











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


