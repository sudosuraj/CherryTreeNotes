# WAF Bypass

A few techniques that consistently get past naive filters/WAFs:

```bash
# 1. Spoof a real browser UA — many WAF rules only trigger on tool UAs (curl, python-requests, etc.)
-H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)"

# 2. Add noise to break pattern-matching: random query param, extra POST field, shuffled header order
?v=<random>   &z=<random-noise-field>

# 3. Full-string URL-encode the whole payload, not just special chars
# e.g. config/app.conf -> %63%6f%6e%66%69%67%2f%61%70%70%2e%63%6f%6e%66

# 4. Split a blocked keyword with string concatenation (works when the filter does a literal substring match)
document['coo'+'kie']    # instead of document.cookie

# 5. Case variation alone is rarely enough on its own — combine with one of the above

# 6. Swap a blocked keyword operator for its symbolic equivalent
OR -> ||     AND -> &&

# 7. Double a blocked keyword so removing it once still leaves a valid one
SSELECTELECT -> (strip "SELECT" once) -> SELECT

# 8. Unicode confusable / homoglyph characters to dodge an exact-string keyword blacklist
exec -> 𝙚𝓍𝓮𝘤   (visually similar, different codepoints — bypasses naive string match, some runtimes still execute it as the real keyword)
```
