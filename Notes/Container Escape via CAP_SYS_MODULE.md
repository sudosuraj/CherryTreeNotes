# Container Escape via CAP_SYS_MODULE

If a container has `CAP_SYS_MODULE`, build and insert a kernel module to run a command on the host as root.

```bash
capsh --print          # or deepce.sh — confirm CAP_SYS_MODULE is held

# module source: constructor calls call_usermodehelper() to spawn a host-level shell
make
insmod rev.ko
```

Distinct from a `--privileged --pid=host` escape or a `/proc/sys/kernel/core_pattern` escape — check for all three capability/config weaknesses independently.
