# JWT Attacks

`alg: none` — some implementations skip signature verification when the header declares no algorithm. Decode the token, change the header, drop the signature, resend.

```bash
# header: {"alg":"none","typ":"JWT"}
# token becomes: base64url(header) + "." + base64url(payload) + "."   (empty signature)
```

Always also try: weak/guessable HMAC secret (crack with `hashcat -m 16500`), and swapping RS256→HS256 using the server's public key as the HMAC secret.
