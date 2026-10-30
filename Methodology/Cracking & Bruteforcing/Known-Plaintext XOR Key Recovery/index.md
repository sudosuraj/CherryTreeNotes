# Known-Plaintext XOR Key Recovery

If you know any fixed part of the plaintext (a flag prefix, a header, a known field), XOR it against the matching ciphertext bytes to recover that many key bytes directly — no brute force needed for a short repeating key.

```python
def xor(a: bytes, b: bytes) -> bytes:
    return bytes(x ^ y for x, y in zip(a, b))

known_plaintext = b"THM{"               # or any known prefix/suffix
key_part = xor(ciphertext[:len(known_plaintext)], known_plaintext)
# extend with more known bytes (suffix, repeated structure) to recover the full key
```
