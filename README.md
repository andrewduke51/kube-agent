# kube-agent

A small teaching project for diagnosing a Kubernetes pod that will not start.
It demonstrates the complete LLM tool-calling loop without MCP: the model asks
for a named tool, Python runs that allow-listed operation, and the model uses
the minimized result to write its diagnosis.

## Safety model

The model receives exactly two tools: `list_pods()` and
`get_pod_events(pod_name)`. Both are fixed to the `agent-demo` namespace.
Python dispatches those names explicitly and runs fixed `kubectl` argument
lists—model output is never executed as a shell command. Only current pod
status and diagnostic event fields are sent to the LLM. Kubeconfig content,
credentials, and other namespaces are never sent.

The demo ServiceAccount has a namespace-scoped Role that can only `get`,
`list`, and `watch` Pods and Events. It has no write verbs, so it cannot modify
workloads.

## Run the demo

Prerequisites: Python 3.11+, `kubectl`, access to the intended cluster, and an
OpenAI API key in `OPENAI_API_KEY`. Do not put the key in a file.

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt

# After reviewing the manifests and confirming the target cluster:
kubectl create namespace agent-demo
kubectl apply -f demo/rbac.yaml
kubectl apply -f demo/app-bad-image.yaml

python agent.py "Why isn't my demo app starting?"
```

Later, change the image in `demo/app-bad-image.yaml` to a valid tag, apply it,
wait for Ready, and rerun the same question. The agent should identify the
current Ready state while treating old pull warnings as historical events.

## Actual sanitized transcript

Pending the live, confirmed-cluster test. This section will contain the exact
question, tools called, image-pull event, and post-fix result with any private
details removed.
