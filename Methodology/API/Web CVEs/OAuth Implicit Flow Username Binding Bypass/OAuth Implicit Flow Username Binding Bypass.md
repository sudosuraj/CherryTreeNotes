# OAuth Implicit Flow Token/Username Binding Bypass

If a custom SSO/OAuth-style login returns the access token directly to the client (implicit grant, no server-side code exchange) and a *separate* username/identity parameter rides along in the same request, the server may trust the username independently of the token instead of verifying the token is actually bound to it:

```
POST /api/login
client_token=<your-own-valid-token>&client_username=administrator
```
Root cause is a check like `if client_token in valid_tokens: session['username'] = client_username` — any valid token plus an arbitrary username impersonates that user. The fix binds them: `valid_tokens[client_token] == client_username`.
