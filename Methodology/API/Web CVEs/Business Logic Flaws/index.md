# Business Logic Flaws

Client-side-only validation: a check implemented in frontend JS with no matching server-side re-check. Always test by hitting the API directly instead of going through the UI.

```bash
# bypass a frontend-only check by calling the API endpoint straight
curl -s -c cookies.txt -b cookies.txt -X POST http://<TARGET>/<api-endpoint> \
  -H "Content-Type: application/json" \
  -d '<payload the UI would normally block>'
```

— TryHackMe / Fools Mate (JS blocked a winning move in `/js/app.js`; `/api/move` accepted it with no server-side check)
