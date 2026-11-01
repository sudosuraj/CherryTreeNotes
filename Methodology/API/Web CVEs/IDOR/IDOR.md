# IDOR

Try adjacent/sequential IDs on any endpoint keyed by an object ID, with no re-check of ownership:

```bash
GET /<resource>/1
GET /<resource>/2
# ... walk the ID space; watch for admin/privileged records at low IDs
```

Also try omitting a filter parameter entirely rather than guessing a value — a query that normally scopes results to "your own" records sometimes returns everything when that parameter is just absent (`?owner=` or no param at all, instead of a specific ID).
