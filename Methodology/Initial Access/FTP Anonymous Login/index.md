# FTP Anonymous Login

Always try anonymous FTP before anything else — it's a free foothold when enabled.

```bash
ftp <TARGET>
# user: anonymous, password: (blank or anything)
ls -la
get <file>
```

