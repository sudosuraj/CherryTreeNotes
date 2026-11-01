# Known Plaintext Attack on Encrypted ZIP

If an encrypted ZIP contains a file whose plaintext content is known (e.g. a standard LICENSE, README, or config), use bkcrack to recover the internal keys and decrypt all files.

```bash
# Install bkcrack: https://github.com/kimci86/bkcrack

# Create a zip of the known plaintext file (must be uncompressed)
zip -r0 plain.zip known_file.txt

# Recover keys
bkcrack -C encrypted.zip -c path/in/zip/known_file.txt -P plain.zip -p known_file.txt

# Decrypt with recovered keys (outputs new zip with chosen password)
bkcrack -C encrypted.zip -k <KEY1> <KEY2> <KEY3> -U decrypted.zip newpassword
unzip -P newpassword decrypted.zip
```

Common known-plaintext sources: `/etc/os-release`, `index.html` with predictable content, LICENSE files, default config files.
