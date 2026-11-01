# Mattermost Internal Channel Credential Harvest

When a help desk ticket system (osTicket) assigns a support email address for ticket updates, any email to that address gets appended to the ticket thread — visible to the ticket holder without authentication. This allows registering accounts on internal services (e.g., Mattermost) that send verification emails to that address.

## Exploit Flow

```
1. Create a guest ticket on osTicket → receive a ticket-update email address
   e.g.: <ticket-id>@helpdesk.<domain>

2. Register on the internal service (Mattermost, etc.) using that ticket email
   → Verification/confirmation email lands in the ticket thread

3. View the ticket status page (no auth required) → see verification link

4. Activate the account and log in

5. Browse internal channels for credentials, secrets, sensitive info
```

## osTicket Ticket Status Check

```bash
# Check ticket status with ticket number + email
curl "http://<TARGET>/support/index.php?a=status" \
  -d "e=<ticket-email>&l=<ticket-number>"
```

## Notes
- Works because osTicket appends all emails to the ticket thread, visible without login
- Mattermost default port: 8065
- Look for passwords in "root" or internal channels (e.g. `#general`, `#it-support`)
- Credentials found in internal chat may have been reused across services/OS users
- Common pattern: employee posts creds or hints in internal chat → lateral movement
