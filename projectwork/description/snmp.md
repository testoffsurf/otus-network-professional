# SNMP (Simple Network Management Protocol)










ip access-list standard SNMP-SnmpAccessHost-ACL
 remark ===[ Allowing access to the SNMP protocol from an IP address: 10.67.1.34 (APP-GRNSEC04) ]===
 permit 10.67.1.34
 remark ===[ We prohibit everything that is not parted, above ]===
 deny   any






