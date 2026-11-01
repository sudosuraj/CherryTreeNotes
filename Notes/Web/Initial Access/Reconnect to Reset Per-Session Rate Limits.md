# Reconnect to Reset Per-Session Rate Limits

Some rate limits are tied to the connection/session, not the account or IP. If a service caps attempts per socket/session (e.g. "3 tries then disconnect"), open a fresh connection for every single guess instead of trying to defeat the limit directly — it resets the counter for free.

**IP-based rate limit bypass via spoofed headers** — if the limit is per source IP and the app trusts a client-supplied header to determine it, inject a different IP each request:

```
X-Forwarded-For: <random-IP>
X-Real-IP: <random-IP>
X-Originating-IP: <random-IP>
Client-IP: <random-IP>
```

Try all four; which one the backend trusts depends on the stack. Rotate the value on every request (Burp Intruder: set the header as a payload position, use a number list or random IP generator). Also works by cycling the `PHPSESSID` cookie on PHP apps that track attempts per session.
