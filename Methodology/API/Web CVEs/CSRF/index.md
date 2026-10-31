# CSRF

If a CSRF token is derived from a predictable value (username, user ID) instead of being random and tied to the session, you can compute a valid token for any known user without ever seeing their real one:

```bash
echo -n "<target-username>" | md5sum   # or whatever hash/derivation the app uses
# use the result as csrf_token in a forged request for that user
```
