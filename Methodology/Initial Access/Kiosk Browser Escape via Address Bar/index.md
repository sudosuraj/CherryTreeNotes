# Kiosk Browser Escape via Address Bar

A locked-down kiosk that still exposes a browser address bar (e.g. via a "help/support" link) can usually be driven to a file:// or local-binary path, breaking out to a shell as the kiosk's own account:

```
<address bar>: C:\windows\system32\cmd.exe
```
If that's blocked, try `file:///C:/Windows/System32/cmd.exe`, a `javascript:` URI, or downloading and launching an arbitrary exe through the browser's own download-and-run flow. Once you have a shell, check the user's home directory for saved tool configs (`putty.conf`, browser profiles) — kiosk accounts often have plaintext creds for whatever management tooling they're meant to use.

The same address-bar trick works on any Windows Open/Save file dialog, not just browsers — any kiosk app that lets you open or save a file gets you a native file-picker, and its path/filename field takes the same payload:
```
<file dialog address/filename bar>: C:\Windows\System32\cmd.exe
```
If the app has no visible "open file" feature at all, check for a right-click context menu with a **Print** option — "Print to PDF" opens a Save dialog as a side effect, which is the same breakout surface.

Once in, if the kiosk accepts a UNC path to "open" a document, point it at your own SMB share serving a payload exe instead of a file — some kiosk apps will fetch and execute it directly.
