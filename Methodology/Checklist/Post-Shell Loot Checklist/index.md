# Post-Shell Loot Checklist

Things to check immediately after landing any shell, before running a full linpeas pass:

```bash
cat *.config *.env *.ini 2>/dev/null          # cleartext creds in app config files, cwd first
find / -iname '*backup*' -o -iname '*.kdbx' -o -iname '*.apk' 2>/dev/null
cat /etc/passwd | grep -v nologin             # other real users to target
cat /etc/hosts                                 # internal hostnames for pivoting
sudo -l                                        # before touching linpeas
grep -ri "password\|secret\|key" /var/www/*/config* 2>/dev/null   # app DB/API creds often reused as OS creds
```

