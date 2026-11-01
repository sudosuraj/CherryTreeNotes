# Cron Script Symlink Leak into World-Readable Log

If a privileged cron/scheduled script blindly `cat`s a file from a directory you can write to (a "staging" or "config" folder) into a log that's world-readable, replacing that file with a symlink leaks whatever target you point it at — root's SSH key, a keytab, anything the cron user can read:

```bash
ln -sf /home/<target-user>/.ssh/id_rsa <staging-dir>/<expected-config-filename>
# wait for the next cron run
cat <world-readable-log-path>     # the symlink target's contents now appear here
```
Read the script first (via any access you already have) to confirm it really does an unguarded `cat "$FILE"` rather than validating the file's type/owner before reading it.
