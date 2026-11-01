# XLSM VBA Macro Credential Extraction

Excel macro-enabled workbooks (`.xlsm`) on SMB shares often contain embedded database or service credentials in VBA code.

Extract VBA without opening Excel:
```bash
olevba "Currency Volume Report.xlsm"
```

Look for hardcoded `Uid=`, `Pwd=`, `Password=`, `ConnectionString=` strings in the macro output.

Use extracted credentials immediately against MSSQL / SMB / WinRM:
```bash
mssqlclient.py '<user>:<pass>'@<TARGET> -windows-auth
nxc smb <TARGET> -u <user> -p <pass>
```
