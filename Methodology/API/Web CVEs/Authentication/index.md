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
