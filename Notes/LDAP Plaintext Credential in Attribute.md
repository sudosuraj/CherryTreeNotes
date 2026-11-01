# LDAP Plaintext Credential in Attribute

With any bind (even a low-priv one on a POSIX-style directory, `uid=...,ou=people,...` rather than AD `cn=...`), dump the whole tree — operators sometimes leave an old plaintext password sitting in a free-text attribute like `description`.

```bash
ldapsearch -x -H ldap://<target> -D "uid=<user>,ou=people,dc=<domain>" -w '<pass>' -b "dc=<domain>" "(objectClass=*)"
```
If the bind DN is wrong (`cn=` guessed instead of `uid=`), the whole search fails — confirm the directory's actual naming convention first (`ou=people` is the POSIX/OpenLDAP default, vs AD's `cn=Users`).
