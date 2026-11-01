# SMB Alternative Data Streams (ADS)

Windows NTFS files can have multiple data streams. Hidden streams on SMB shares may contain passwords or sensitive data. Use `smbclient` to detect and read them.

## Detect ADS via smbclient
```bash
smb: \path\> allinfo "filename.txt"
# Look for: stream: [:StreamName:$DATA], N bytes
```

## Read ADS Content
```bash
smb: \path\> get "filename.txt:StreamName"
# Downloads stream content as a local file
```

## Full Session
```bash
smbclient //<TARGET>/<share> -U <user>
smb: \> cd "path/to/directory"
smb: \path\> allinfo "suspicious_file.txt"
smb: \path\> get "suspicious_file.txt:Password"
```

## Windows CMD (if you have a shell)
```cmd
# List streams
dir /r filename.txt
# Read stream
more < "filename.txt:StreamName"
# Or:
type "filename.txt:StreamName"
```

## Notes
- Common streams: `:Password`, `:Zone.Identifier`, custom names
- Zero-byte file + non-zero named stream = hidden data
- `allinfo` in smbclient shows both `::$DATA` (main) and named streams
