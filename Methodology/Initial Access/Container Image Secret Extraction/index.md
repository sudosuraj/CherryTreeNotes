# Container Image Secret Extraction

If a running app's container image is pullable (public registry, or creds you already have), pull it and read the source/config directly instead of trying to extract it over HTTP.

```bash
docker pull <registry>/<image>:<tag>
docker run -it --rm <registry>/<image>:<tag> /bin/sh
# inside: read app source, env defaults, config files for hardcoded keys/secrets
cat /app/*.py /app/config* 2>/dev/null
```
