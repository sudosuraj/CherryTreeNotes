# KeePass (.kdbx) Cracking

```bash
keepass2john <file>.kdbx > kdbx.hash
john --wordlist=/usr/share/wordlists/rockyou.txt kdbx.hash
# then open the .kdbx with the cracked master password
```

— TryHackMe / Silent Monitor
