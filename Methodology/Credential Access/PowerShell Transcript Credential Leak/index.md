# PowerShell Transcript Credential Leak

PowerShell transcript logs record all commands typed in a session, including passwords passed as arguments. Check `PSTranscripts` directories for accidentally logged credentials.

## Find Transcript Files
```powershell
# Look in hidden directories on C:\
gci -recurse -force -file C:\PSTranscripts
gci -recurse -force -file C:\ -include "PowerShell_transcript*.txt" 2>$null

# Common locations:
# C:\PSTranscripts\
# C:\Users\<user>\Documents\WindowsPowerShell\
# C:\Windows\Temp\
```

## Read Transcript
```powershell
type C:\PSTranscripts\<date>\PowerShell_transcript.<host>.<random>.txt
```
Search for credential patterns:
```powershell
Select-String -Path "C:\PSTranscripts\*\*.txt" -Pattern "password|pass|cred" -CaseSensitive:$false
```

## Bash (if you have file access)
```bash
grep -i "password\|pass\|cred" PSTranscripts/**/*.txt
```

## Notes
- Transcripts enabled via Group Policy: `Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell > Turn on PowerShell Transcription`
- Check `net use` commands — often contain credentials: `net use \\server\share /user:domain\user password`
- Also check event log 4104 (ScriptBlock logging) if you have event log access
