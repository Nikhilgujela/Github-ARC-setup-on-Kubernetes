upgrade this into a **production-grade ARC setup** with:

* 🔐 Secure authentication (GitHub App instead of PAT)
* ⚡ Proper autoscaling (min/max runners)
* 📊 Observability (logs + metrics ideas)
* 🧹 Clean architecture diagram
* 🚀 Best-practice configuration

---

# 🚀 Production-Grade GitHub ARC Setup on Kubernetes

---

# 🧠 Architecture (Production View)

```text id="arch1"
GitHub Org / Repo
        │
        ▼
GitHub App (recommended auth)
        │
        ▼
ARC Controller (arc-systems)
        │
        ▼
AutoscalingRunnerSet (arc-runners)
        │
        ├── Listener Pod (always 1)
        │
        └── Runner Pods (scale up/down dynamically)
                │
                ▼
        GitHub Actions Jobs
```

---

# 🔐 1. Authentication (IMPORTANT UPGRADE)

## ❌ Avoid PAT in production

PAT issues:

* expires
* security risk
* limited control

---

## ✅ Use GitHub App (BEST PRACTICE)

### Create GitHub App:

Go to:

```text id="g1"
GitHub → Settings → Developer settings → GitHub Apps
```

Create app with:

### Permissions:

* Repository: Read & Write
* Actions: Read & Write
* Metadata: Read

---

### Install app on:

* your repo OR org

---

### You get:

```text id="g2"
App ID
Installation ID
Private Key
```

---

# ⚙️ 2. Install ARC Controller

```bash id="c1"
helm install arc \
  --namespace arc-systems \
  --create-namespace \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

---

# ⚙️ 3. Install Runner Scale Set (Production)

## Create `values.yaml`

```yaml id="v1"
githubConfigUrl: https://github.com/<ORG>/<REPO>

githubConfigSecret:
  github_app_id: "<APP_ID>"
  github_app_installation_id: "<INSTALLATION_ID>"
  github_app_private_key: |
    -----BEGIN RSA PRIVATE KEY-----
    ...
    -----END RSA PRIVATE KEY-----

runnerGroup: default

minRunners: 1
maxRunners: 5

template:
  spec:
    containers:
      - name: runner
        image: ghcr.io/actions/actions-runner:latest
        command: ["/home/runner/run.sh"]
    serviceAccountName: arc-runner
```

---

## Install:

```bash id="c2"
helm upgrade --install arc-runner-set \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set \
  -n arc-runners \
  --create-namespace \
  -f values.yaml
```

---

# ⚡ 4. Autoscaling Behavior

| Situation  | Result                 |
| ---------- | ---------------------- |
| No jobs    | 1 runner (warm)        |
| Job queued | scale up automatically |
| Load high  | up to maxRunners       |
| Idle       | scale back down        |

---

# 📊 5. Monitoring (Production Essential)

## Check ARC status

```bash id="m1"
kubectl get pods -n arc-systems
kubectl get pods -n arc-runners
```

---

## Watch scaling live

```bash id="m2"
kubectl get pods -n arc-runners -w
```

---

## Controller logs

```bash id="m3"
kubectl logs -n arc-systems deployment/arc-gha-rs-controller --tail=200
```

---

# 🧪 6. GitHub Workflow Test

```yaml id="wf1"
name: ARC Production Test

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: arc-runner-set
    steps:
      - name: Check runner
        run: echo "Running on ARC production setup 🚀"
```

---

# 🧹 7. Clean Best Practices

## ✔ Always set:

```text id="bp1"
minRunners = 1   → warm capacity
maxRunners = 5+  → burst scaling
```

---

## ✔ Use GitHub App instead of PAT

| Feature          | PAT   | GitHub App |
| ---------------- | ----- | ---------- |
| Security         | ❌ low | ✅ high     |
| Expiry           | ❌ yes | ✅ no       |
| Production ready | ❌ no  | ✅ yes      |

---

## ✔ Separate environments

```text id="bp2"
arc-runner-dev
arc-runner-prod
```

---

# 🚨 Common Production Mistakes

### ❌ Using PAT in production

→ causes Pending / silent failures

---

### ❌ No minRunners

→ cold start delays

---

### ❌ Wrong runnerGroup

→ jobs not assigned properly

---

# 🎯 Final Production Summary

✔ ARC Controller manages lifecycle
✔ Listener stays always ON
✔ Runner pods scale automatically
✔ GitHub App = secure authentication
✔ No manual runner installation needed

---
