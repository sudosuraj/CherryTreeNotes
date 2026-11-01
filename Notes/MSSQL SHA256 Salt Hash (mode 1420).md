# MSSQL SHA256 Salt Hash (mode 1420)

Web applications backed by MSSQL sometimes store passwords as `sha256($pass.$salt)` — hashcat mode 1420. The format in the DB is typically: `<32-byte-hex-hash>:<hex-salt>`.

```bash
# hashcat format: hash:salt (both hex, salt appended to password before hashing)
echo '<hash_hex>:<salt_hex>' > hash.txt
hashcat -a0 -m1420 hash.txt /usr/share/wordlists/rockyou.txt

# verify: sha256(password + unhex(salt)) = hash
python3 -c "import hashlib,binascii; print(hashlib.sha256(b'<PASS>'+binascii.unhexlify('<SALT>')).hexdigest())"
```

Extract from MSSQL dump:
```sql
SELECT username, password_hash, password_salt FROM users;
-- password_hash and password_salt are usually stored as varbinary — convert to hex for hashcat
SELECT username, CONVERT(NVARCHAR(MAX), password_hash, 2), CONVERT(NVARCHAR(MAX), password_salt, 2) FROM users;
```
