# GitHub Secret Search (Commit History)

If a target's source is publicly hosted, search GitHub code search for leaked secrets, and check commit history for ones that were "removed" but never rotated — a removed value is still in the git log.

```bash
# on GitHub: search for the app's distinctive strings/filenames to find the repo
# once found:
git clone <repo-url>
git log --all -p | grep -iE "key|secret|password|token"   # search every commit's diff, not just HEAD
```
A commit message like "fix: remove key" is itself a strong hint — the previous commit still has it.
