# LaTeX Injection (\lstinputlisting File Read)

Any user-supplied input compiled into a LaTeX document (an "equation renderer", invoice generator, etc.) is a command-injection surface if you can break out of inline math mode:

```latex
$ $ \lstinputlisting{/etc/passwd} $ $
```
Use the same primitive to read app source and config (`.htpasswd`, `.htaccess`, backend PHP) before looking for a jump to actual command execution — LaTeX has packages (`\write18`, if shell-escape is enabled) that run shell commands directly, worth testing too.
