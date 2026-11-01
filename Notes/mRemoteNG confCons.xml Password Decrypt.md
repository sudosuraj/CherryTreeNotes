# mRemoteNG confCons.xml Password Decrypt

mRemoteNG stores saved connection passwords AES-encrypted in `confCons.xml`:
```
%AppData%\mRemoteNG\confCons.xml
```

Look for `Password=` attributes in the XML — values are base64-encoded AES ciphertext.

Decrypt with:
```bash
# Python tool
python3 mremoteng_decrypt.py confCons.xml

# Java tool (older)
java -jar decipher_mremoteng.jar <ciphertext_base64>
```

Default encryption key is `mR3m` — if no master password is set the tool decrypts without prompting.
