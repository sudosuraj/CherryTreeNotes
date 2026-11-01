# Outlook PST Email Credential Extraction

Outlook `.pst` (Personal Storage Table) and `.ost` files contain emails, contacts, and calendar data. Credentials are often found in email threads on Windows targets.

## Extract PST on Linux

```bash
# Install readpst
apt install readpst

# Convert PST to mbox format (creates directory with .mbox files)
readpst -o /tmp/mail_out/ Archive.pst

# Browse extracted mail
ls /tmp/mail_out/
cat /tmp/mail_out/Inbox.mbox

# Or use mutt to read mbox files
mutt -f /tmp/mail_out/Inbox.mbox
```

## Encrypted ZIP Containing PST

```bash
# Standard unzip fails on AES-256 encrypted zips
# Use 7z instead
7z x archive.zip
# Enter password when prompted

# After extraction, use readpst
readpst extracted.pst
```

## Extract .mdb Access Database (for passwords)

```bash
# Install mdbtools
apt install mdbtools

# List all tables
mdb-tables backup.mdb

# Export a specific table (look for auth_user, USERINFO, etc.)
mdb-export backup.mdb auth_user
mdb-export backup.mdb USERINFO | head -20

# Quick grep for passwords across all tables
for table in $(mdb-tables backup.mdb); do
  mdb-export backup.mdb "$table" 2>/dev/null | grep -i "pass\|pwd" && echo "TABLE: $table"
done
```

## Notes
- `.pst` = Microsoft Outlook data file; `.ost` = offline cached copy
- Passwords found in `.mdb` Access databases often reused as zip passwords or system passwords
- Check tables: `auth_user`, `USERINFO`, `employees`, `acc_user` for credential data
- `readpst` outputs to mbox format readable by mutt/thunderbird or plain text search
