# 🚀 Setup ARC Controller in Kubernetes Using Helm

## 📦 Install ARC Controller

```bash
NAMESPACE="arc-systems"

helm install arc \
  --namespace "${NAMESPACE}" \
  --create-namespace \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

---

## 🔍 Check Controller Status

```bash
kubectl get all -n arc-systems
```

### Example Output

```text
NAME                                         READY   STATUS    RESTARTS   AGE
pod/arc-gha-rs-controller-7c668d7847-pjrld   1/1     Running   0          28s

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/arc-gha-rs-controller   1/1     1            1           28s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/arc-gha-rs-controller-7c668d7847   1         1         1       28s
```

---

## 🔍 Check Pods Only

```bash
kubectl get pods -n arc-systems
```

### Example Output

```text
NAME                                     READY   STATUS    RESTARTS   AGE
arc-gha-rs-controller-7c668d7847-pjrld   1/1     Running   0          42s
```

---

# 🔐 GitHub PAT Token Setup

Create a Personal Access Token:

```text
GitHub → Settings → Developer settings → Personal access tokens
→ Fine-grained personal access token
```

---

# ⚙️ Configure Runner Scale Set

## 📦 Install Runner Scale Set

```bash
INSTALLATION_NAME="arc-runner-set"
NAMESPACE="arc-runners"
GITHUB_CONFIG_URL="https://github.com/<your_org_or_repo>"
GITHUB_PAT="<PAT>"

helm install "${INSTALLATION_NAME}" \
  --namespace "${NAMESPACE}" \
  --create-namespace \
  --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
  --set githubConfigSecret.github_token="${GITHUB_PAT}" \
  oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

---

## 📋 Verify Helm Releases

```bash
helm list -A
```

---

## 📊 Check Autoscaling Runner Set

```bash
kubectl get autoscalingrunnersets -n arc-runners
```

```bash
kubectl get ephemerarunnersets -n arc-runners
kubectl get ephemerarunners -n arc-runners
```

---

## 🧠 Verify Listener Pod

After setup, you should see the listener in `arc-systems`:

```bash
kubectl get pods -n arc-systems
```

### Example Output

```text
NAME                                     READY   STATUS    RESTARTS   AGE
arc-gha-rs-controller-7c668d7847-pjrld   1/1     Running   0          22m
arc-runner-set-d7ffb588-listener         1/1     Running   0          92s
```

---

## 🏃 Watch Runner Pods

```bash
kubectl get pods -n arc-runners -w
```

---

# 🎯 Summary

* ARC Controller runs in `arc-systems`
* Runner Scale Set runs in `arc-runners`
* Listener pod is created automatically
* Runner pods are created only when GitHub jobs run

---

If you want, I can also convert this into a **GitHub README with architecture diagram + troubleshooting section**.
