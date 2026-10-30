# Rule-based Wordlist + Service Bruteforce

If recon gives a plausible password hint (a year, a word on the site, a naming pattern), mutate it with hashcat rules instead of brute-forcing blind.

```bash
echo "<hinted-word>" > base.txt
hashcat --stdout base.txt -r /usr/share/hashcat/rules/dive.rule > wordlist.txt
hydra -l <USER> -P wordlist.txt <TARGET> <service>
```

— TryHackMe / Operation Promotion
