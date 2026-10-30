# JWT Attacks

`alg: none` — some implementations skip signature verification when the header declares no algorithm. Decode the token, change the header, drop the signature, resend.

```bash
# header: {"alg":"none","typ":"JWT"}
# token becomes: base64url(header) + "." + base64url(payload) + "."   (empty signature)
```

Always also try: weak/guessable HMAC secret (crack with `hashcat -m 16500`), and swapping RS256→HS256 using the server's public key as the HMAC secret.

If `none` and the RS256→HS256 swap both get rejected, try the reverse direction too — some validators pick the verification method by the claims/key available rather than strictly enforcing the header's declared algorithm (algorithm confusion isn't one-directional).
