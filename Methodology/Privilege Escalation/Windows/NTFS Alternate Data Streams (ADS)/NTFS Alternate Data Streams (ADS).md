# NTFS Alternate Data Streams (ADS)

NTFS files can have multiple named data streams. Data can be hidden in ADS, including flags or credentials stored in `file.txt:streamname`.

## List ADS on a File
```cmd
dir /r file.txt
# Shows: file.txt:streamname:$DATA
```

## Read ADS Content
```cmd
more < file.txt:streamname
# or
type file.txt:streamname
```

## PowerShell
```powershell
Get-Item -Path .\file.txt -Stream *
Get-Content -Path .\file.txt -Stream streamname
```

## Find ADS Recursively
```cmd
dir /r /s C:\path\
```

## Create ADS (Hiding Data)
```cmd
echo "hidden content" > visible.txt:hidden
```

## Notes
- Common CTF trick: flag hidden in `root.txt:DATA` stream
- ADS are not shown by default `dir` — must use `/r`
- `type` command reads only the default stream; use `more <` to read ADS
- SMB shares preserve ADS; HTTP downloads do not
