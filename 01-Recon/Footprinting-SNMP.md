# Footprinting — SNMP

```bash
snmpwalk -v2c -c public <IP>
onesixtyone -c community.txt <IP>     # брут community-стрінгів
snmpbulkwalk -v2c -c public <IP>
```
**Що шукати:** користувачі, процеси, мережеві інтерфейси, інколи паролі в команд-лайнах.
