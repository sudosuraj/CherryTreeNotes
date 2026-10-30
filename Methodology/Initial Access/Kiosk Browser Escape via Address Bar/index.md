# Kiosk Browser Escape via Address Bar

A locked-down kiosk that still exposes a browser address bar (e.g. via a "help/support" link) can usually be driven to a file:// or local-binary path, breaking out to a shell as the kiosk's own account:

```
<address bar>: C:\windows\system32\cmd.exe
```
If that's blocked, try `file:///C:/Windows/System32/cmd.exe`, a `javascript:` URI, or downloading and launching an arbitrary exe through the browser's own download-and-run flow. Once you have a shell, check the user's home directory for saved tool configs (`putty.conf`, browser profiles) — kiosk accounts often have plaintext creds for whatever management tooling they're meant to use.
