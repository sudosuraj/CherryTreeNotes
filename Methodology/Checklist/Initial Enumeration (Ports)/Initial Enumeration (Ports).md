# Initial Enumeration (Ports)

## Fast full port scan
```bash
nmap -p- --min-rate 5000 -oA nmap/allports <IP>
```

## Detailed scan on open ports
```bash
nmap -sC -sV -p <ports> -oA nmap/detailed <IP>
```

## UDP (top 20)
```bash
nmap -sU --top-ports 20 <IP>
```

## Common port → what to do
| Port | Go to |
|------|-------|
| 21 | FTP — try anon login, check version CVEs |
| 22 | SSH — try found creds, key files |
| 23 | Telnet — try default creds |
| 25/587 | SMTP — user enum, phishing pivot |
| 53 | DNS — zone transfer: `dig axfr @<IP> <domain>` |
| 80/443 | Web App checklist |
| 110/143 | POP3/IMAP — try found creds |
| 135/139/445 | SMB checklist |
| 389/636 | LDAP — enum AD |
| 1433 | MSSQL — try default sa, xp_cmdshell |
| 1521 | Oracle DB |
| 2049 | NFS — showmount -e <IP> |
| 3306 | MySQL — try root no pass, found creds |
| 3389 | RDP — try found creds |
| 5985/5986 | WinRM — try found creds with evil-winrm |
| 6379 | Redis — try unauth |
| 8080/8443 | Alt web — same as web checklist |
| 11211 | Memcached |
| 27017 | MongoDB — try unauth |

## SMB quick checks
```bash
crackmapexec smb <IP>
smbclient -L //<IP> -N
smbmap -H <IP>
enum4linux-ng <IP>
```

## NFS
```bash
showmount -e <IP>
mount -t nfs <IP>:/share /mnt/nfs
```

## FTP
```bash
ftp <IP>   # try anonymous / anonymous
# if login works: get all files, check for writable dirs
```
