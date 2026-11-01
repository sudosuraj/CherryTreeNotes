# JWT Attacks

`alg: none` — some implementations skip signature verification when the header declares no algorithm. Decode the token, change the header, drop the signature, resend.

```bash
# header: {"alg":"none","typ":"JWT"}
# token becomes: base64url(header) + "." + base64url(payload) + "."   (empty signature)
```

Always also try: weak/guessable HMAC secret (crack with `hashcat -m 16500`), and swapping RS256→HS256 using the server's public key as the HMAC secret.

If `none` and the RS256→HS256 swap both get rejected, try the reverse direction too — some validators pick the verification method by the claims/key available rather than strictly enforcing the header's declared algorithm (algorithm confusion isn't one-directional).

"Psychic Signatures" (CVE-2022-21449) — some Java ECDSA (ES256) implementations fail to reject a degenerate all-zero signature, so it verifies against *any* payload with no key needed:

```bash
# signature (r=0, s=0) DER-encoded, base64url: MAYCAQACAQA
<base64url(header)>.<base64url(payload)>.MAYCAQACAQA
```
Only affects vulnerable Java versions (15–18 before the fix) using ECDSA, but worth a quick try whenever you see `"alg":"ES256"` on a Java-backed service.

Trusted-registry/JWKS forgery — if the app fetches its JWKS from a URL rather than using a pinned key, and that URL is reachable/redirectable to you (SSRF, DNS, a compromised upstream host), host your own JWKS and sign tokens with the matching private key.
