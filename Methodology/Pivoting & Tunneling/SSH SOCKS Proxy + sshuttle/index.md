# SSH SOCKS Proxy + sshuttle

Lighter-weight alternatives to Ligolo-ng when you already have SSH access — no agent binary to upload.

```bash
# dynamic SOCKS proxy through an existing SSH session
ssh -D 9050 -i <key> <user>@<pivot-host>
proxychains -q nmap -sT -Pn -p <ports> <internal-range> --open

# or, full-subnet VPN-style tunnel in one command (needs sshuttle installed locally)
sshuttle --dns -vr <user>@<pivot-host> <internal-range> -x <pivot-host> --ssh-cmd 'ssh -i <key>'
```
`sshuttle` routes all your traffic to the target range transparently — no `proxychains` prefix needed on subsequent commands.
