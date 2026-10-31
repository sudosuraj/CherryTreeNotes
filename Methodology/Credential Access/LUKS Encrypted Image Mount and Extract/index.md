# LUKS Encrypted Image Mount and Extract

LUKS (Linux Unified Key Setup) encrypted disk images found on targets can be cracked offline with hashcat, then mounted to reveal contents.

## Crack LUKS Password

```bash
# Extract LUKS hash for cracking
python3 luks2john.py backup.img > luks.hash
# or: bruteforce-luks -f /usr/share/wordlists/rockyou.txt backup.img

# Crack with hashcat (mode 14600 = LUKS)
hashcat -m 14600 luks.hash /usr/share/wordlists/rockyou.txt
```

## Mount Decrypted Image

```bash
# Open LUKS container (prompts for passphrase)
cryptsetup open --type luks backup.img <name>
# Creates /dev/mapper/<name>

# Verify device
ls -l /dev/mapper/

# Mount the decrypted filesystem
mkdir -p /mnt/<name>
mount /dev/mapper/<name> /mnt/<name>/

# Browse contents
find /mnt/<name>/ -type f
```

## Cleanup

```bash
umount /mnt/<name>/
cryptsetup close <name>
```

## Notes
- LUKS images sometimes found via SMB shares or FTP with anonymous access
- Common contents: config files, backup keys, web app configs with hardcoded credentials
- `bruteforce-luks` is slow; hashcat GPU cracking via luks2john is much faster
- Mount requires root (or sudo) on attacker machine
