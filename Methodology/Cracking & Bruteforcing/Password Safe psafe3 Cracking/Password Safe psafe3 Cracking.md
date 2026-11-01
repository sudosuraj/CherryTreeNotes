# Password Safe (.psafe3) Cracking

Password Safe databases (`.psafe3`) use a strong KDF but are offline-crackable via john or hashcat.

```bash
# extract the hash
pwsafe2john <file>.psafe3 > psafe3.hash

# crack with john
john psafe3.hash --wordlist=/usr/share/wordlists/rockyou.txt

# hashcat mode 5200 (Password Safe v3)
hashcat -a0 -m5200 psafe3.hash /usr/share/wordlists/rockyou.txt
```

Once cracked, open the database with Password Safe (Linux: `pwsafe`, or via Wine with the Windows binary) to extract all stored credentials.
