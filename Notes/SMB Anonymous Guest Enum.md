# SMB Anonymous/Guest Enum

```bash
nxc smb <TARGET> -u guest -p '' --shares
smbclient.py guest:''@<TARGET>
# If IPC$ is readable as Guest, enumerate local users via RID brute-force:
nxc smb <TARGET> -u guest -p '' --rid
```

