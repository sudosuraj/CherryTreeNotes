# MDB Access Database Credential Extraction

Microsoft Access `.mdb` files often store credentials in plaintext or weakly encrypted tables.

**Install mdbtools (Linux):**
```bash
apt install mdbtools
```

**List tables:**
```bash
mdb-tables backup.mdb
```

**Dump a table:**
```bash
mdb-export backup.mdb auth_user
mdb-export backup.mdb <tablename>
```

**Export all tables:**
```bash
for table in $(mdb-tables backup.mdb); do
    mdb-export backup.mdb "$table" > "$table.csv"
done
```

**Common useful table names:** `Users`, `auth_user`, `Accounts`, `Credentials`, `Config`

**Access password encryption:** older `.mdb` files use simple XOR — tools like `mdb-export` decode them automatically.

**Also check FTP/anonymous shares** for `.mdb` files — commonly found alongside backup files.
