# SSH Key Recovery from Git History

Developers sometimes accidentally commit SSH keys or credentials, then remove them in a follow-up commit. The key remains in the repository history and can be recovered with git.

## Find Interesting Commits

```bash
# Review commit history with file names changed
git log --name-only --oneline

# Look for:
# - "reverted accidental commit" followed by an earlier commit adding keys/creds
# - commits modifying .key, .pem, id_rsa, credentials files
```

## Extract Key from Old Commit

```bash
# Option 1: Check diff between two commits
git diff <old_commit> <new_commit> -- path/to/key.file

# Option 2: Checkout the file from the specific commit
git checkout <commit_hash> -- resources/integration/authcredentials.key
cat resources/integration/authcredentials.key

# Option 3: Show file contents at a specific commit
git show <commit_hash>:path/to/file
```

## Search All History for Secrets

```bash
# Search all commits for patterns
git log --all --full-history --diff-filter=A -- "*.key" "*.pem" "*id_rsa*"

# Use git-secrets or trufflehog for automated scanning
trufflehog git file://. --only-verified
```

## Notes
- Always check for commit messages with "revert", "remove", "fix accidental", "cleanup keys"
- The "deleted" file still exists in the object store until gc runs
- Restore the repo to its original state after extraction (`git checkout HEAD -- <file>`)
