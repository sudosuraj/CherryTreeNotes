# WSL Bash History Credential Leak

When WSL (Windows Subsystem for Linux) is installed, `bash.exe` runs as the Windows user's associated Linux root.  
Check for it when you have a low-priv Windows shell:

```cmd
where /R c:\ bash.exe
```

The system32 shortcut often doesn't exist; the real binary is buried in WinSxS:
```cmd
C:\Windows\WinSxS\amd64_microsoft-windows-lxss-bash_*\bash.exe
```

Run it to drop into the WSL root shell, then read bash history:
```bash
cat /root/.bash_history
```

Credentials or SSH commands left in history often reveal lateral movement paths or admin creds.
