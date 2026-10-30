# SSRF

A hostname allow-list is not an IP restriction — if the allowed hostname itself resolves to loopback/internal (via `/etc/hosts` or attacker-controlled DNS), it's a loopback bypass, not a real allow-list.

```bash
http://<TARGET>/<proxy-endpoint>?url=http://<allowed-hostname>/admin
http://<TARGET>/<proxy-endpoint>?url=http://<allowed-hostname>/<sensitive-path>
```

Chain to watch for: a second check further in (e.g. `if not request.remote_addr.startswith("127.")`) can be satisfied by the exact same SSRF, since the proxying server is now the one making the request.

— TryHackMe / Operation Coldstart (`ALLOWED_HOSTS = {"kestrel.thm"}`, but `kestrel.thm` → 127.0.0.1 on the box; reached `/admin/notes` which leaked creds)
