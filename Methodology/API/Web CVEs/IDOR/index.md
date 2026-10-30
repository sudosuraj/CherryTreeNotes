# IDOR

Try adjacent/sequential IDs on any endpoint keyed by an object ID, with no re-check of ownership:

```bash
GET /<resource>/1
GET /<resource>/2
# ... walk the ID space; watch for admin/privileged records at low IDs
```
