# Cisco IOS Type 7 Password Decrypt

Type 7 is a reversible XOR cipher using the static Cisco key `tfd;kfoA,.iyewrkldJKD`.  
Found in router/switch configs as: `username <x> password 7 <hex>`

Decrypt the hex string:
```bash
# Python one-liner
python3 -c "
key='tfd;kfoA,.iyewrkldJKD'
h='0242114B0E143F015F5D1E161713'
off=int(h[:2]); data=bytes.fromhex(h[2:])
print(''.join(chr(b ^ ord(key[(off+i)%len(key)])) for i,b in enumerate(data)))
"
```

Many online tools also decode it (e.g. `https://www.ifm.net.nz/cookbooks/passwordcracker.html`).

Type 5 (MD5-crypt `$1$`) requires hashcat:
```bash
hashcat -m 500 -a 0 cisco_type5.hash rockyou.txt
```

Type 8 / Type 9 require hashcat modes 9200 / 9300.
