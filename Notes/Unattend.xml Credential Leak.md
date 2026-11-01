# Unattend.xml Credential Leak

Windows deployment/sysprep answer files left behind at `C:\Windows\Panther\Unattend.xml` (or `sysprep.inf`/`sysprep.xml` nearby) routinely contain a base64-encoded admin/service account password from the original image build:

```cmd
type C:\Windows\Panther\Unattend.xml
```
Decode the `<Password><Value>` field (plain base64, often with a `AdministratorPassword` or similarly-named XML element) with CyberChef or `base64 -d`. Always check this path on any fresh Windows box before assuming no creds are available locally.
