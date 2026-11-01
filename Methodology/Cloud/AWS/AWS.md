# AWS

Public S3 bucket enumeration — no credentials needed, try this on any discovered bucket name/hostname:

```bash
aws s3 ls s3://<bucket-name> --no-sign-request
curl http://<bucket>.s3-website-<region>.amazonaws.com/<object-key>
```
A 404 with an XML `NoSuchKey`/`NoSuchBucket` body confirms you've got a real S3-backed endpoint even before listing works.
