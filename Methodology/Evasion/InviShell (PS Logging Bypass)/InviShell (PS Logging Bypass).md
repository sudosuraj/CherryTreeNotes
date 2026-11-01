# InviShell (PS Logging Bypass)

Bypass PowerShell ScriptBlock logging, Module logging, and transcription by hooking CLR (Common Language Runtime) before PowerShell loads.

## Usage

### Non-admin (registry-based)
```cmd
C:\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

### Admin (direct)
```cmd
C:\Tools\InviShell\RunWithPathAsAdmin.bat
```

## What It Bypasses
- PowerShell ScriptBlock Logging (Event ID 4104)
- PowerShell Module Logging (Event ID 4103)
- PowerShell Transcription logging
- AMSI (Antimalware Scan Interface) for PowerShell

## Mechanism
InviShell injects a DLL into the PowerShell process that hooks `EtwEventWrite` and AMSI functions before the PowerShell engine initializes, preventing telemetry from being sent.

## Notes
- Must be run before importing malicious modules
- Works on PowerShell 5.1 (Windows PowerShell); less effective on PS 7+
- Combine with AMSI bypass for full coverage
- Does not bypass Constrained Language Mode (CLM) — use AppLocker bypass for that
