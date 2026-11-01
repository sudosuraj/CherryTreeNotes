# SMTP VRFY User Enumeration

The SMTP `VRFY` command asks a server to verify a recipient address without sending mail. Many servers still respond with valid/invalid distinctions useful for username enumeration.

```bash
nc <TARGET> 25
VRFY root
VRFY admin@<domain>
# 250 = exists, 550 = doesn't exist, 252 = cannot verify (but treats as exists)

# automated via smtp-user-enum
smtp-user-enum -M VRFY -U /usr/share/wordlists/metasploit/unix_users.txt -t <TARGET>

# also try EXPN (expands mailing list aliases) and RCPT TO (sends a test recipient)
smtp-user-enum -M RCPT -U users.txt -D <domain> -t <TARGET>
```

Found valid usernames → combine with a password spray or credential stuffing against SMB/SSH/webapp.

Also check `EHLO` response for enabled extensions and open relay:
```bash
nc <TARGET> 25
EHLO test
MAIL FROM:<test@test.com>
RCPT TO:<test@external.com>    # if 250, server is an open relay
```
