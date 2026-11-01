# Session Hunting (SessionHunter)

Find where privileged users (DA, EA) are currently logged in via remote registry queries.

## Find DA Sessions
```powershell
# Using Invoke-SessionHunter (PowerView)
Invoke-SessionHunter -Domain <DOMAIN>
```
Queries remote registry `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList` on each machine to find logged-in users.

## Find Specific User Sessions
```powershell
Invoke-SessionHunter -Domain <DOMAIN> -User <username>
```

## PowerView Alternative
```powershell
Find-DomainUserLocation -UserName <username>
Find-DomainUserLocation -UserGroupIdentity "Domain Admins"
```

## Notes
- Remote registry service must be running on target (default: manual start, sometimes auto)
- Requires at least local admin on targets to read HKLM keys
- Add-RemoteRegBackdoor (RACE.ps1) can enable this without needing admin
- Combine with OverPass-the-Hash to jump to the machine where DA session found
