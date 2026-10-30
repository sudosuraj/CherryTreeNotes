# Cracking & Bruteforcing

Offline hash cracking and online service brute-forcing, by target type.

SNMPv3 is often overlooked in password-reuse sprays — if you have a leaked/breach password list and a set of usernames, spray it before assuming SNMP is a dead end:
```bash
./legba snmp3 --target <IP> --username users.txt --password passwords.txt
snmpwalk -v3 -u <user> -A '<password>' <IP> -l authNoPriv   # authNoPriv community strings/creds sometimes sit in the MIB tree itself
```

If a password policy or onboarding doc leaks the *format* of a password (e.g. a redacted example like `ProtoTr****1973!`), build a hashcat mask instead of a dictionary attack:
```bash
hashcat -O -m <mode> -a3 -1='?u?l?d' 'ProtoTr?1?1?1?11973!' --username hash.txt
```

Cisco "type 5" (MD5-crypt) secrets show up often in router/switch config backups found on shares or in email attachments — don't skip them as uncrackable just because they look like a hash blob:
```bash
hashcat -m500 -a0 cisco_type5.hash <WORDLIST>
```
