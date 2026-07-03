
# 🚀 GitHub Actions Runner Controller (ARC) Setup on Kubernetes

This guide explains how to install and configure **GitHub Actions Runner Controller (ARC)** using Helm on Kubernetes.

---

# 🧱 Architecture Overview

```text id="arch1"
GitHub Repository
        ↓
ARC Controller (arc-systems namespace)
        ↓
AutoscalingRunnerSet (arc-runners namespace)
        ↓
Listener Pod (created automatically)
        ↓
Runner Pods (created only when GitHub job runs)
```

---

# ⚙️ 1. Install ARC Controller

## 📦 Create namespace and install controller

```bash id="c1"
NAMESPACE="arc-systems"

helm install arc \
  --namespace "${NAMESPACE}" \
  --create-namespace \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

---

## 🔍 Verify controller installation

```bash id="c2"
kubectl get all -n arc-systems
```

### Example output

```text id="out1"
NAME                                         READY   STATUS    AGE
pod/arc-gha-rs-controller-xxxx               1/1     Running   1m

NAME                                    READY   UP-TO-DATE   AVAILABLE
deployment.apps/arc-gha-rs-controller   1/1     1            1
```

---

# 🔐 2. Create GitHub PAT Token

Go to:

```text id="pat1"
GitHub → Settings → Developer Settings → Personal Access Tokens
```

### Required permissions:

For repository runners:

* `repo`
* `workflow`

For organization runners:

* `admin:org`

---

# ⚙️ 3. Install Runner Scale Set

## 📦 Deploy ARC runners

```bash id="c3"
INSTALLATION_NAME="arc-runner-set"
NAMESPACE="arc-runners"
GITHUB_CONFIG_URL="https://github.com/<owner>/<repo>"
GITHUB_PAT="<YOUR_PAT>"
```

```bash id="c4"
helm install "${INSTALLATION_NAME}" \
  --namespace "${NAMESPACE}" \
  --create-namespace \
  --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
  --set githubConfigSecret.github_token="${GITHUB_PAT}" \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

---

# 📊 4. Verify Installation

## Check Helm releases

```bash id="c5"
helm list -A
```

---

## Check Autoscaling Runner Set

```bash id="c6"
kubectl get autoscalingrunnersets -n arc-runners
```

Example:

```text id="out2"
NAME              STATE
arc-runner-set    Running
```

---

# 🧠 5. Understand ARC Resources

## Important resources in ARC v0.14+

| Resource               | Purpose                      |
| ---------------------- | ---------------------------- |
| `AutoscalingRunnerSet` | Defines runner configuration |
| Listener Pod           | Connects to GitHub           |
| Runner Pods            | Execute CI jobs              |

---

### ❌ NOT USED in this version:

```text id="wrong"
ephemerarunnersets ❌ (does not exist in v0.14+)
ephemerarunners ❌ (removed/renamed in newer versions)
```

---

# 👀 6. Check Listener Pod

After successful setup:

```bash id="c7"
kubectl get pods -n arc-systems
```

Example:

```text id="out3"
arc-gha-rs-controller-xxxx   1/1 Running
arc-runner-set-xxxx-listener 1/1 Running
```

---

# 🏃 7. Watch Runner Pods

```bash id="c8"
kubectl get pods -n arc-runners -w
```

---

# 🧪 8. Test Runner (GitHub Workflow)

Create workflow:

```yaml id="wf1"
name: ARC Test

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: arc-runner-set
    steps:
      - run: echo "Hello from ARC 🚀"
```

---

# 🎯 Expected Behavior

When workflow runs:

```text id="flow2"
GitHub Job triggered
        ↓
Listener receives job
        ↓
ARC creates runner pod
        ↓
Runner executes job
        ↓
Pod is deleted after completion
```

---

# 🚨 Common Issues

## ❌ AutoscalingRunnerSet stuck in Pending

* Invalid PAT
* Wrong GitHub URL
* Missing repo access

---

## ❌ No runner pods created

* No workflow triggered
* Wrong `runs-on` label

---

## ❌ Listener not created

* GitHub authentication failure

---

# ✅ Final Summary

* ARC Controller runs in `arc-systems`
* Runner Scale Set runs in `arc-runners`
* Listener is auto-created by controller
* Runner pods appear only when jobs run
* No manual runner installation required

---
