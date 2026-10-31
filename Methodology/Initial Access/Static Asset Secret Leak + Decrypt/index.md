# Static Asset Secret Leak + Decrypt

Check JS/CSS/static assets for hardcoded keys before treating an encrypted backup/config file as a dead end.

```bash
grep -riE "key|secret|iv" /path/to/static/*.js
# then, if it's AES-ECB with a known key:
python3 -c "
from Crypto.Cipher import AES
key = b'<key-from-js>'
data = open('config.enc','rb').read()
print(AES.new(key, AES.MODE_ECB).decrypt(data))
"
```
