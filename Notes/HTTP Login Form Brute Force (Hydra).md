# HTTP Login Form Brute Force (Hydra)

```bash
hydra -L <users.txt or -l USER> -P <wordlist> <TARGET> http-post-form \
  '<path>:<user-field>=^USER^&<pass-field>=^PASS^:<failure-string-on-page>'
```

`-I` ignores a previous restore file; get `<failure-string-on-page>` exactly right or every attempt reads as a hit.
