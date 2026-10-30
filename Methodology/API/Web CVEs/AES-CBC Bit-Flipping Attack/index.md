# AES-CBC Bit-Flipping Attack

CBC with no MAC/integrity check is malleable: flipping a bit in ciphertext block N flips the same bit position in the *decrypted plaintext of block N+1* (while corrupting block N's own plaintext). Useful when you control some of the plaintext and know the target value in the next block.

```python
known_char = 'b'      # what's currently at that position in the next block's plaintext
target_char = 'a'      # what you want it to decrypt to

flip = ord(known_char) ^ ord(target_char)
ciphertext_bytes[pos] ^= flip   # pos = same byte offset, but in the PRECEDING block
```
Place your controlled input so the byte you want to flip lands at the right offset in the block before your target — often means padding with filler so the target string starts exactly at a block boundary.
