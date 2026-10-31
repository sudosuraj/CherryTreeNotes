# Steghide Extraction

Extract hidden data from image files (JPEG, BMP, WAV, AU) using steghide. Often used in CTF scenarios to hide SSH keys or passwords.

## Extract Without Passphrase
```bash
steghide extract -sf <image.jpg>
# If no passphrase: just press Enter
```

## Extract With Known Passphrase
```bash
steghide extract -sf <image.jpg> -p "<passphrase>"
```

## Check if Data is Embedded
```bash
steghide info <image.jpg>
```

## Brute Force Passphrase
```bash
stegseek <image.jpg> /usr/share/wordlists/rockyou.txt
# Or:
for pass in $(cat wordlist.txt); do
  steghide extract -sf image.jpg -p "$pass" 2>/dev/null && echo "Found: $pass" && break
done
```

## Other Steg Tools
```bash
# Check all layers of PNG/JPEG
binwalk -e image.jpg
exiftool image.jpg
strings image.jpg | grep -i key
# zsteg for PNG LSB
zsteg image.png
```

## Notes
- Try blank passphrase first — very common
- Extracted file may be a private key, password, or flag
- Combine with `ssh2john` if extracted file is encrypted SSH key
