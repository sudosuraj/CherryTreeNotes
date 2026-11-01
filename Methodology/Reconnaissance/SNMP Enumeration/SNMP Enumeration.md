# SNMP Enumeration

SNMP UDP/161 is often missed in TCP-only scans. Always scan UDP on targets.

```bash
# Detect SNMP (UDP scan required):
nmap -sU -p 161 --min-rate 5000 <TARGET>

# Full SNMPv1/v2c walk with public community string:
snmpwalk -v 2c -c public <TARGET> | tee snmp_data

# Bulk walk (faster):
snmpbulkwalk -v 2c -c public <TARGET> > snmp_data

# Human-readable OIDs (requires mibs):
sudo apt install snmp-mibs-downloader
# Add "mibs +" to /etc/snmp/snmp.conf

# Try other community strings:
onesixtyone -c /opt/SecLists/Discovery/SNMP/snmp.txt <TARGET>
```

**High-value OIDs:**
```
sysDescr.0          — OS / software versions
sysContact.0        — email address (username enumeration)
sysName.0           — hostname / app info
hrSWRunName         — running process names
hrSWRunParameters   — process command-line args (can expose credentials)
ifDescr             — network interfaces
ipAddrTable         — IP addresses
```

Often reveals internal hostnames, credentials in process args, software versions, and usernames for brute-forcing.
