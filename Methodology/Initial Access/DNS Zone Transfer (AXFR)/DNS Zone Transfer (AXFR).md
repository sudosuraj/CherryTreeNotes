# DNS Zone Transfer (AXFR)

Always try this against any in-scope authoritative DNS server — a misconfigured server will hand over its entire zone, including internal/unlisted subdomains.

```bash
dig axfr <domain> @<dns-server-ip>
```
