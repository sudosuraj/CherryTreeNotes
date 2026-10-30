# LocalAccountTokenFilterPolicy Remote Admin Bypass

A freshly-added local admin account often can't actually use WinRM/remote admin tools yet — by default, UAC remote restrictions filter a local (non-built-in-Administrator) account's token down to standard-user rights over the network, even though it's in the Administrators group. Flip the policy key (needs an existing SYSTEM/admin-equivalent foothold to set it):

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```
After this, `evil-winrm`/remote admin access with that local account's creds works as expected.
